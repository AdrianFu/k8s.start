# Setup system configuration for k8s

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

# Setup containerd Using Official Binaries

More detail on:
https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/install-containerd-on-ubuntu-22-04.html

## Install containerd

First, download the latest version of containerd from GitHub and extract the files to the /usr/local/ directory
```
wget https://github.com/containerd/containerd/releases/download/v1.6.2/containerd-1.6.2-linux-amd64.tar.gz
or
wget https://github.com/containerd/containerd/releases/download/v1.6.15/containerd-1.6.15-linux-amd64.tar.gz

sudo tar Czxvf /usr/local containerd-1.6.2-linux-amd64.tar.gz
```

Then, download the systemd service file and set it up so that you can manage the service via systemd.
```
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mv containerd.service /usr/lib/systemd/system/
```

Finally, start the containerd service
```
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

check the status of the containerd service
```
sudo systemctl status containerd
```

## Install runC

runC is an open-source container runtime for spawning and running containers on Linux according to the OCI specification.

Download the latest version of runC from GitHub and install it as /usr/local/sbin/runc
```
wget https://github.com/opencontainers/runc/releases/download/v1.1.1/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```


## Containerd configuration for Kubernetes

containerd uses a configuration file config.toml for handling its demons. When installing containerd using official binaries, you will not get the configuration file. So, generate the default configuration file using the below commands.
```
sudo mkdir -p /etc/containerd/
containerd config default | sudo tee /etc/containerd/config.toml
```

If you plan to use containerd as the runtime for Kubernetes, configure the systemd cgroup driver for runC.
```
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

restart the containerd service.
```
sudo systemctl restart containerd
```

# Setup k8s

Disable swap as required by k8s
```
sudo swapoff -a
```

Install additional packages
```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
```

Setup package repository for k8s
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Configure respository
```
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
```

Install k8s packages
```
sudo apt-get install -y kubelet=1.24.10-00 kubeadm=1.24.10-00 kubectl=1.24.10-00
```

Make sure these packages are not automatically updated - maintain manual control
```
sudo apt-mark hold kubelet kubeadm kubectl
```

# Setup Cluster

Initialize cluster - On Control Plane node
```
sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.24.10
```

Setup kube config to interactive cluster with kubectl
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Config networking plugin
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Create token for workers to join cluster
```
kubeadm token create --print-join-command
```

# Notes

## Firewall enabled in Ubuntu 22
Verify
```
sudo ufw status verbose
```

To disable
```
sudo ufw disable

sudo reboot
```

Or Open Port
```
sudo ufw allow 22/tcp
```

open http and https service ports
```
sudo ufw allow http
sudo ufw allow https
```

## List all pods and their nods

```
kubectl get pod -o=custom-columns=NODE:.spec.nodeName,NAME:.metadata.name --all-namespaces
```