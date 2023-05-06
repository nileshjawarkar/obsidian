### 1) [Installer docker](https://docs.docker.com/engine/install/)
### 2) [Install microk8s](https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#2-deploying-microk8s)
### 3) Enable metallb - Loadbalancer
- Define IP rage to be used by LB. This is usually some IP address from network used for current kubernetes nodes (VM's).
- For example - 192.168.200.10-192.168.200.15
- microk8s enable metallb
### 4) [Join node](https://microk8s.io/docs/clustering)
### 5) Tests
- microk8s kubectl get all --all-namespaces
- microk8s kubectl get nodes
