# Kubernetes :- The Production grade container orchestration  Engine 
## Kubernetes multinode setup 
###  we have 3 machines; 1 master and 2 worker nodes
## Pre-requisite 


 ## Installing  docker and kubeadm in all the nodes 
 ```
 [root@master ~]# yum  install  docker kubeadm  -y
 ```
 ## if kubeadm is not present in your repo 
 you can browse this link [kubernetes repo](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)  <br/>
 
## yum can be configured by running this command 
```
cat  <<EOF  >/etc/yum.repos.d/kube.repo
[kube]
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
gpgcheck=0
EOF
```

## You may have to change docker cgroup driver from overlay to Systemd  (since kubernetes 1.20)
 
```
cat  <<X  >/etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

X
``` 
 
## Start service of docker & kubelet in all the nodes 
 ```
 systemctl enable --now  docker kubelet
 ```
## for kernel configuration
```
modprobe br_netfilter
```
```
echo '1' > /proc/sys/net/ipv4/ip_forward
```
 ## Do this only on Kubernetes Master 
 We are here using Calico Networking, so we need to pass some parameter 
 you can start [Kubernetes_networking](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) from this  <br/>
 
```
kubeadm  init --pod-network-cidr=192.168.0.0/16
```
## this is optional 
### In case of cloud services like aws, azure if want to bind public with certificate of kubernetes 
```
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=0.0.0.0   --apiserver-cert-extra-sans=publicip,privateip,serviceip
```

## Note: IF you want to bind your controlplane with public IP also 

```
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=0.0.0.0   --apiserver-cert-extra-sans=publicip,privateip,serviceip  --control-plane-endpoint=publicIP
```

### Use the output of above command and paste it to all the worker nodes

## Do this step in master node 
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

##  Now apply calico project 
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```
## Or Download 3.25.0 latest version as now october 2020 

```
wgetcurl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml 
kubectl apply -f calico.yaml

```

### Now 3.20 installation has been changed

```
 kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.0/manifests/tigera-operator.yaml
 kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.0/manifests/custom-resources.yaml
 kubectl get pods -n calico-system
 
```

# INstalling calico in minikube cluster..

```
minikube start --network-plugin=cni --cni=calico

```

## ADding new node in minikube 

```
  minikube node add
```

After this all nodes will be in ready state

## Now you can check nodes status
```
[root@master ~]# kubectl get nodes
NAME                 STATUS   ROLES    AGE     VERSION
master.example.com   Ready    master   11m     v1.12.2
node1.example.com    Ready    <none>   9m51s   v1.12.2
node2.example.com    Ready    <none>   9m25s   v1.12.2
node3.example.com    Ready    <none>   9m3s    v1.12.2
```

Good luck guys !!
