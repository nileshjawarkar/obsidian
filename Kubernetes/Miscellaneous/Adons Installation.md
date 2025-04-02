## 1) Metallb - Load balancer

### a) kind

#### Install
``` sh
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb --namespace metallb-system --create-namespace
```

#### Get IP address range to be used for load balancer - 

1) Using docker : 
``` sh
docker network inspect -f '{{.IPAM.Config}}' kind

# Output
[{172.18.0.0/16  172.18.0.1 map[]} {fc00:f853:ccd:e793::/64  fc00:f853:ccd:e793::1 map[]}]
```
- The output will contain a cidr such as 172.18.0.0/16. We want our loadbalancer IP range to come from this subclass. 

2) OR using the your network config
	On my VM network, i used following IP rage 192.168.200.2-192.168.200.255. Out of these IP's 192.168.200.2-192.168.200.20 are reserved as a static IP's for my vm's. So I can use 192.168.200.21-192.168.200.255 for load balancing

#### Use IP Range -

We can configure MetalLB to use 172.18.255.200 to 172.18.255.250 by creating the IPAddressPool

``` Yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: example
  namespace: metallb-system
spec:
  addresses:
  - 172.18.255.200-172.18.255.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: empty
  namespace: metallb-system
```
- Save above yaml as metallb-config.yaml
- Create IPAddressPool using `kubectl apply -f metallb-config.yaml`

### b) Microk8s

- Define IP rage to be used by LB. Please refer to _"Get IP address range section"_.
- `microk8s enable metallb`
- Above command will install metallb as well as it will ask IR range to configure it.

## 2) Ingress - Deployment mode (microk8s will install it in demon mode)

``` sh
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

**Important links**

1) [Ingress-nginx deployment](https://kubernetes.github.io/ingress-nginx/deploy/)

## 3) Calico - Network plugin

``` sh
helm repo add projectcalico https://docs.tigera.io/calico/charts
helm install calico projectcalico/tigera-operator --version v3.25.1 --namespace tigera-operator --create-namespace
```

