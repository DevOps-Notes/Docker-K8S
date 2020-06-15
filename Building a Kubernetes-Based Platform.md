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
  
  

