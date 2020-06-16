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

  

  

  

