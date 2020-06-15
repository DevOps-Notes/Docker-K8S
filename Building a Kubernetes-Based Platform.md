# Building a Kubernetes-Based Platform

* Servers

  | R     | I           | C                                                            | Size                                |
  | ----- | ----------- | ------------------------------------------------------------ | ----------------------------------- |
  | M     | 192.168.1.4 | kube-apiserver<br/>kube-controller-manager<br/>kube-scheduler<br/>etcd | Standard A3 (4 vcpus, 7 GiB memory) |
  | Ma    | 192.168.1.5 | kube-apiserver<br/>kube-controller-manager<br/>kube-scheduler<br/>etcd | Standard A3 (4 vcpus, 7 GiB memory) |
  | No    | 192.168.1.6 | kubelet<br/>kube-proxy<br/>docker<br/>etcd                   | Standard A3 (4 vcpus, 7 GiB memory) |
  | Node2 | 192.168.1.7 | kubelet<br/>kube-proxy<br/>docker                            | Standard A3 (4 vcpus, 7 GiB memory) |
  | lb    | 192.168.1.8 | NGINX Layer 4                                                | Standard A3 (4 vcpus, 7 GiB memory) |
  | lb    | 192.168.1.9 | NGINX Layer 4                                                | Standard A3 (4 vcpus, 7 GiB memory) |

  

