# Instructions from cloud guru

It is broken.
For proper instructions, follow **Setup k8s on Ubuntu 20 or 22.md**

# Host File

## To add entries

1. Edit
   ```
   sudo vi /etc/hosts
   ```

1. Then inside vi
   ```
   172.31.36.113 k8s-control
   172.31.36.210 k8s-worker1
   172.31.40.92  k8s-worker2
   ```

1. Log out and back in

# Host Name

## To change host name to k8s-control
```
sudo hostnamectl set-hostname k8s-control
```

# Setup containerd

## To enable modules on startup
```
cat << EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

## To enable modules now
```
sudo modprobe overlay
sudo modprobe br_netfilter
```

# Setup system configuration for k8s

## Setup networking for k8s on startup
```
cat << EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

## To apply changes immediately
```
sudo sysctl --system
```

# To install containerd

1. Install
   ```
   sudo apt-get update && sudo apt-get install -y containerd
   ```

2. Create configuraiton file
   ```
   sudo mkdir -p /etc/containerd
   sudo containerd config default | sudo tee /etc/containerd/config.toml
   sudo systemctl restart containerd
   sudo systemctl status containerd
   ```

# To setup k8s
1. Disable swap as required by k8s
   ```
   sudo swapoff -a
   ```
1. Install additional packages
   ```
   sudo apt-get update && sudo apt-get install -y apt-transport-https curl
   ```
1. Setup package repository for k8s
   ```
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   ```
1. Configure respository
```
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
```

1. Install k8s packages
   ```
   sudo apt-get install -y kubelet=1.24.0-00 kubeadm=1.24.0-00 kubectl=1.24.0-00
   ```

1. Make sure these packages are not automatically updated - maintain manual control
   ```
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

1. Repeat on work nodes and other control plain nodes.

# To setup Cluster
1. Initialize cluster - On Control Plane node
   ```
   sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.24.0
   ```
   If above does not work as planned, then probably need to install different containerd.
   
   For further info: https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/install-containerd-on-ubuntu-22-04.html

1. Setup kube config to interactive cluster with kubectl
   ```
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

1. Config networking plugin
   ```
   kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
   ```

1. Create token for workers to join cluster
   ```
   kubeadm token create --print-join-command
   ```

1. On each work node, run the 'kubeadm join ...' command printed out in last step.