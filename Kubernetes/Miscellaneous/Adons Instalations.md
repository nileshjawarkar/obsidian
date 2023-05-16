### 1) Metallb - Load balancer

- Define IP rage to be used by LB. This is usually some IP address from network used for current kubernetes nodes (VM's).
- For example - 192.168.200.10-192.168.200.15
- microk8s enable metallb

### 2) Ingress - Deployment mode (microk8s will install in demon mode)

``` sh
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

**Important links**

1) [Ingress-nginx deployment](https://kubernetes.github.io/ingress-nginx/deploy/)

