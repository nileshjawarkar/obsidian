A service provides a single point of access from outside the Kubernetes cluster and allows you to dynamically access a group of replica pods. There are several ways to expose your service to the outside of your Kubernetes cluster, and you'll want to select the appropriate one based on your specific use case.

The four main options we'll be comparing here are: **ClusterIP**, **NodePort**, **LoadBalancer**, and **Ingress**. Each provides a way to expose services and is useful in different situations.

1) _For internal application access within a Kubernetes cluster,_ ClusterIP is the preferred method. It is a default setting in Kubernetes and uses an internal IP address to access the service.
2) _To expose a service to external network requests_, NodePort, LoadBalancer, and Ingress are possible options. 

- NodePort type service exposes port on the cluster node to provide access to the service with no load balancing capabilities.
- LoadBalancer type service provides load balancer and it  is a extension of NodePort service.
- Problem with LoadBalancer service is that it need load balancer for each service, on the other hand Ingress can use single ingress resource with different routing rules to expose multiple services. Which make it less resource intensive.

![[Pasted image 20230515192641.png]]


### Installation  &  Configuration

- If load balancer is not installed, then we need to define the port forwarding from master
``` sh

kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
```
- Please check Installation instruction for Ingress and Metallb load balancer at [[m2 - Adons Instalations]]

- Check object created after install
``` sh
kubectl get all -n ingress-nginx

# Output

# PODS
############################
NAME                                           READY   STATUS    RESTARTS   AGE
pod/ingress-nginx-controller-f6c55fdc8-jz256   1/1     Running   0          158m

# Services
#############################
NAME                                         TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
service/ingress-nginx-controller-admission   ClusterIP      10.152.183.180   <none>           443/TCP                      158m
service/ingress-nginx-controller             LoadBalancer   10.152.183.222   192.168.200.11   80:31103/TCP,443:30245/TCP   158m

# Deployement
##########################
NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           158m

# Replica-set
###########################
NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-f6c55fdc8   1         1         1       158m
```

### Create Ingress

1) Create deployment
``` sh
kubectl create deployment demo1 --image=httpd --port=80 --replicas=2
kubectl create deployment demo2 --image=nginx --port=80 --replicas=2
```

2) Create service
``` sh
kubectl expose deployment demo1 --port=8080 --target-port=80 --name demo1-service
kubectl expose deployment demo2 --port=8080 --target-port=80 --name demo2-service
```

3) Create Ingress

- Generate yaml using CLI
``` sh
kubectl create ingress demo-ingress --class=nginx  \
--annotation "nginx.ingress.kubernetes.io/rewrite-target"="/" \
--rule="demo.localdev.me/*=demo1-service:8080" \
--rule="demo.localdev.me/nginx=demo2-service:8080" \
--dry-run=client -o yaml

# Output

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  creationTimestamp: null
  name: demo-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: demo.localdev.me
    http:
      paths:
      - backend:
          service:
            name: demo1-service
            port:
              number: 8080
        path: /
        pathType: Exact
      - backend:
          service:
            name: demo2-service
            port:
              number: 8080
        path: /nginx
        pathType: Exact
status:
  loadBalancer: {}
```

- Re-run  the command with out _--dry-run=client -o yaml_ to define the ingress rule
``` yaml
kubectl create ingress demo-ingress --class=nginx  \
--annotation "nginx.ingress.kubernetes.io/rewrite-target"="/" \
--rule="demo.localdev.me/*=demo1-service:8080" \
--rule="demo.localdev.me/nginx=demo2-service:8080"
```

4) Check Ingress details
``` sh
kubectl describe ingress demo-ingress

# output

Name:             demo-ingress
Labels:           <none>
Namespace:        default
Address:          192.168.200.11
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host              Path  Backends
  ----              ----  --------
  demo.localdev.me  
                    /        demo1-service:8080 (10.1.161.218:80,10.1.174.32:80)
                    /nginx   demo2-service:8080 (10.1.161.219:80,10.1.174.33:80)
Annotations:        nginx.ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    22s (x2 over 29s)  nginx-ingress-controller  Scheduled for sync
```

5) Tests - Check if we can access services using host + endpoints

-  Add host name "demo.localdev.me" and IP address of ingress (Shown above) to the   "/etc/hosts" file
``` Text
192.168.200.11 demo.localdev.me
```

- Test 01
``` sh
curl demo.localdev.me

# output

<html><body><h1>It works!</h1></body></html>
```

- Test 02
``` sh
curl demo.localdev.me/nginx

# output

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```


