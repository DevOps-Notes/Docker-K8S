# Building a Kubernetes-Based Platform

* Servers

  | Role     | IP           | Components                                                            | Size                                |
  | ----- | ----------- | ------------------------------------------------------------ | ----------------------------------- |
  | master1 | 192.168.1.4 | kube-apiserver<br/>kube-controller-manager<br/>kube-scheduler<br/>etcd | Standard A3 (4 vcpus, 7 GiB memory) |
  | master2 | 192.168.1.5 | kube-apiserver<br/>kube-controller-manager<br/>kube-scheduler<br/>etcd | Standard A3 (4 vcpus, 7 GiB memory) |
  | node1 | 192.168.1.6 | kubelet<br/>kube-proxy<br/>docker<br/>etcd                   | Standard A3 (4 vcpus, 7 GiB memory) |
  | node2 | 192.168.1.7 | kubelet<br/>kube-proxy<br/>docker                            | Standard A3 (4 vcpus, 7 GiB memory) |
  | lb1 | 192.168.1.8 | NGINX Layer 4                                                | Standard A3 (4 vcpus, 7 GiB memory) |
  | lb2 | 192.168.1.9 | NGINX Layer 4                                                | Standard A3 (4 vcpus, 7 GiB memory) |

* Initialize the servers

  Xshell - Tools - Send Key Input To All Session

  ```sh
  systemctl stop firewalld
  systemctl disable firewalld

  setenforce 0
  sed -i 's/enforcing/disabled/' /etc/selinux/config
  
  # Turn off swap if Physical Servers
  swapoff -a
  vim /etc/fstab
  
  vim /etc/hosts
  192.168.1.4 master1
  192.168.1.5 master2
  192.168.1.6 node1
  192.168.1.7 node2
  192.168.1.8 lb1
  192.168.1.9 lb2
  
  cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
  ntpdate time.windows.com
  ```
  
  ```sh
  hostnamectl set-hostname master1
  hostnamectl set-hostname master2
  hostnamectl set-hostname node1
  hostnamectl set-hostname node2
  hostnamectl set-hostname lb1
  hostnamectl set-hostname lb2
  ```
  
  

* Security certificate

  * CA: Certificate Authority
  * CSR: Certificate Signing Request
  * CRT: Certificate
  * -key: Certificate Key
  * PEM: Privacy Enhanced Mail
  * TLS: Transport Layer Security

  ```sh
  # Get cfssl tools
  wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
  wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
  wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
  chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
  mv cfssl_linux-amd64          /usr/local/bin/cfssl
  mv cfssljson_linux-amd64      /usr/local/bin/cfssljson
  mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
  
  # Create folders, 2 sets of certificates are required
  mkdir -p ~/TLS/{etcd,k8s}
  cd TLS/etcd
  
  # Create CA: ca.pem & ca-key.pem
  cat > ca-config.json << EOF
  {
      "signing":{
          "default":{
              "expiry":"87600h"
          },
          "profiles":{
              "www":{
                  "expiry":"87600h",
                  "usages":[
                      "signing",
                      "key encipherment",
                      "server auth",
                      "client auth"
                  ]
              }
          }
      }
  }
  EOF
  cat > ca-csr.json << EOF
  {
      "CN":"etcd CA",
      "key":{
          "algo":"rsa",
          "size":2048
      },
      "names":[
          {
              "C":"CN",
              "L":"Beijing",
              "ST":"Beijing"
          }
      ]
  }
  EOF
  cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
  ls *pem
  
  # Create Etcd HTTPS certificate using self-signed CA: server.pem & server-key.pem
  cat > server-csr.json << EOF
  {
      "CN":"etcd",
      "hosts":[
          "192.168.1.4",
          "192.168.1.5",
          "192.168.1.6"
      ],
      "key":{
          "algo":"rsa",
          "size":2048
      },
      "names":[
          {
              "C":"CN",
              "L":"BeiJing",
              "ST":"BeiJing"
          }
      ]
  }
  EOF
  cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server
  ls server*pem
  ```

* Deploy etcd cluster

  ```sh
  # On etcd-1 / 192.168.1.4 / master1
  wget https://github.com/etcd-io/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz
  mkdir /opt/etcd/{bin,cfg,ssl} -p
  tar zxf etcd-v3.4.9-linux-amd64.tar.gz
  mv etcd-v3.4.9-linux-amd64/{etcd,etcdctl} /opt/etcd/bin/
  
  cat > /opt/etcd/cfg/etcd.conf << EOF
  #[Member]
  ETCD_NAME="etcd-1"
  ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
  ETCD_LISTEN_PEER_URLS="https://192.168.1.4:2380"
  ETCD_LISTEN_CLIENT_URLS="https://192.168.1.4:2379"
  #[Clustering]
  ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.1.4:2380"
  ETCD_ADVERTISE_CLIENT_URLS="https://192.168.1.4:2379"
  ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.1.4:2380,etcd-2=https://192.168.1.5:2380,etcd-3=https://192.168.1.6:2380"
  ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
  ETCD_INITIAL_CLUSTER_STATE="new"
  EOF
  
  cp ~/TLS/etcd/ca*pem ~/TLS/etcd/server*pem /opt/etcd/ssl/
  
  cat > /usr/lib/systemd/system/etcd.service << EOF
  [Unit]
  Description=Etcd Server
  After=network.target
  After=network-online.target
  Wants=network-online.target
  [Service]
  Type=notify
  EnvironmentFile=/opt/etcd/cfg/etcd.conf
  ExecStart=/opt/etcd/bin/etcd                 \
  --cert-file=/opt/etcd/ssl/server.pem         \
  --key-file=/opt/etcd/ssl/server-key.pem      \
  --peer-cert-file=/opt/etcd/ssl/server.pem    \
  --peer-key-file=/opt/etcd/ssl/server-key.pem \
  --trusted-ca-file=/opt/etcd/ssl/ca.pem       \
  --peer-trusted-ca-file=/opt/etcd/ssl/ca.pem  \
  --logger=zap
  Restart=on-failure
  LimitNOFILE=65536
  [Install]
  WantedBy=multi-user.target
  EOF
  
  # Copy the files to the rest etcd servers (execute line by line)
  scp -r /opt/etcd/ root@192.168.1.5:/opt/
  scp /usr/lib/systemd/system/etcd.service root@192.168.1.5:/usr/lib/systemd/system/
  scp -r /opt/etcd/ root@192.168.1.6:/opt/
  scp /usr/lib/systemd/system/etcd.service root@192.168.1.6:/usr/lib/systemd/system/
  ```

  ```sh
  # On etcd-2 / 192.168.1.5 / master2
  vim /opt/etcd/cfg/etcd.conf
  #[Member]
  ETCD_NAME="etcd-2"
  ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
  ETCD_LISTEN_PEER_URLS="https://192.168.1.5:2380"
  ETCD_LISTEN_CLIENT_URLS="https://192.168.1.5:2379"
  #[Clustering]
  ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.1.5:2380"
  ETCD_ADVERTISE_CLIENT_URLS="https://192.168.1.5:2379"
  ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.1.4:2380,etcd-2=https://192.168.1.5:2380,etcd-3=https://192.168.1.6:2380"
  ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
  ETCD_INITIAL_CLUSTER_STATE="new"
  
  # On etcd-3 / 192.168.1.6 / node1
  vim /opt/etcd/cfg/etcd.conf
  #[Member]
  ETCD_NAME="etcd-3"
  ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
  ETCD_LISTEN_PEER_URLS="https://192.168.1.6:2380"
  ETCD_LISTEN_CLIENT_URLS="https://192.168.1.6:2379"
  #[Clustering]
  ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.1.6:2380"
  ETCD_ADVERTISE_CLIENT_URLS="https://192.168.1.6:2379"
  ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.1.4:2380,etcd-2=https://192.168.1.5:2380,etcd-3=https://192.168.1.6:2380"
  ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
  ETCD_INITIAL_CLUSTER_STATE="new"
  ```

  ```sh
  # Start etcd cluster (server by server)
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
  
  # Verify the cluster
  ETCDCTL_API=3 /opt/etcd/bin/etcdctl --cacert=/opt/etcd/ssl/ca.pem --cert=/opt/etcd/ssl/server.pem --key=/opt/etcd/ssl/server-key.pem --endpoints="https://192.168.1.4:2379,https://192.168.1.5:2379,https://192.168.1.6:2379" endpoint health
  ```

  





