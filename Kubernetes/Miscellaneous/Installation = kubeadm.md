
## [Site](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### Note - If creating cluster using VM's, please make sure all VM has static ip's


## Instruction for Redhat

### Pre-reqs

- A compatible Linux host. The Kubernetes project provides generic instructions for Linux distributions based on Debian and Red Hat, and those distributions without a package manager.
- 2 GB or more of RAM per machine (any less will leave little room for your apps).
- 2 CPUs or more.
- Full network connectivity between all machines in the cluster (public or private network is fine).
- Unique hostname, MAC address, and product_uuid for every node. See here for more details.
- Certain ports are open on your machines. See here for more details.
- Swap disabled. You MUST disable swap in order for the kubelet to work properly.

### Disable swap from /etc/fstab

### Verify the MAC address and product_uuid are unique for every node 
- You can get the MAC address of the network interfaces using the command ip link or ifconfig -a
- The product_uuid can be checked by using the command sudo cat /sys/class/dmi/id/product_uuid

### configure docker

```
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

### Letting iptables see bridged traffic
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

### configure docker

```
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```
## Install kubeadm
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# Set SELinux in permissive mode (effectively disabling it)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```
### Stepes we see till now are comman for master and worker nodes
---

## Prepare master 

### select your pod network addon. I am using callico
- [URL](https://docs.projectcalico.org/getting-started/kubernetes/quickstart)

### start kubeadm
- sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.56.10
- apiserver-advertise-address : IP of local vm
- pod-network-cidr : IP range required for callico.

### Some config for current user
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### Install callico
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

OR 

kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```

### Check Callico is ready
```
watch kubectl get pods -n calico-system
```

### Remove tents on master to scedule pods on it
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

---

## Join worknodes
- print join command on master using following command
```
kubeadm token create --print-join-command
```
- now execute this command on worker nodes.

---

### Get all the nodes on cluster
```
kubectl get nodes -o wide
```
