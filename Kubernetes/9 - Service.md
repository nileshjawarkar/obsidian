
Service provides single IP address and domain (gateway) for multiple replicas of the pod (endpoints) and serve as a load balancer and help pod's to scale.

![[Pasted image 20230504223729.png]]

Lets discuss some terminologies : 
- Service  - Gateway that forward request to the endpoint
- Endpoint - This is individual replica of the pod

#### Create a service and manually configure it ...

**1) Create frontend pod**
``` sh
kubectl run frontend-pod --image ubuntu:latest --command -- sleep 3600
```

**2) Create backend pods**
``` sh
kubectl run endpoint01-pod --image nginx:latest
kubectl run endpoint02-pod --image nginx:latest
```

**3) Test connection between frontend and endpoints**

- _Get pod details_
``` sh
kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
frontend-pod     1/1     Running   0          80s   10.1.174.22   ubuntusrv02   <none>           <none>
endpoint01-pod   1/1     Running   0          63s   10.1.174.23   ubuntusrv02   <none>           <none>
endpoint02-pod   1/1     Running   0          58s   10.1.174.24   ubuntusrv02   <none>           <none>
```

- _Connect to frontend and install curl_
``` sh
kubectl exec -it frontend-pod -- bash

# Then execute apt-get update & apt-get install curl
```

- _From inside frontend - Execute curl on IP of both the end points_
``` sh
curl 10.1.174.23
curl 10.1.174.24
```

Above tests shows that frontend pod and connect with both the endpoints.

**3) Create service**

- _Use following  "nginx-service.yaml" to create service_
``` yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```

- _Execute `kubectl apply -f <FILENAME>`  to create service_
- _Check the service_
``` sh
kubectl get service

# output -
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.152.183.1    <none>        443/TCP    5d8h
nginx-service   ClusterIP   10.152.183.54   <none>        8080/TCP   9s
```

``` sh
kubectl describe service nginx-service

# Output
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.152.183.54
IPs:               10.152.183.54
Port:              <unset>  8080/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```
Please take a note of Endpoints property.. its empty currently.

**4) Add endpoint to the service**

- _Create nginx-endpoints.yaml_
``` yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: nginx-service
subsets:
  - addresses:
      - ip: 10.1.174.23
      - ip: 10.1.174.24
    ports:
      - port:  80
```
 Note - Name of service and endpoint must be same
 
- _Execute `kubectl apply -f <FILENAME>`  to add the endpoints_
- _Check if endpoints added to the service_
``` sh
kubectl describe service nginx-service

# Output
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.152.183.54
IPs:               10.152.183.54
Port:              <unset>  8080/TCP
TargetPort:        80/TCP
Endpoints:         10.1.174.23:80,10.1.174.24:80
Session Affinity:  None
Events:            <none>
```

- _Check if  frontend can access IP of service_
``` sh
kctl exec frontend-pod -- curl 10.152.183.54:8080

# Output
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
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   615  100   615    0     0   118k      0 --:--:-- --:--:-- --:--:--  150k
```

**5) Cleanup**
``` sh
kubectl delete endpoint nginx-service
kubectl delete service nginx-service
kubectl delete pod frontend-pod
kubectl delete pod endpoint01-pod
kubectl delete pod endpoint02-pod
```

#### Create service using yaml file

**1) _nginx-service-all.yaml_- Uses deployment to create pods. In service definition, it uses selector to add pods to the service.**
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
   name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-srv-pod
  template:
    metadata:
      labels:
        app: nginx-srv-pod
    spec:
      containers:
      - name: nginx
        image: nginx:stable
        ports:
        - containerPort: 80 
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-srv-pod
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```

**2) Execute `kubectl apply -f <FILENAME>`  to add the endpoints**

**3) Check the service**
``` sh
kubectl describe service nginx-service

# Output
ame:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx-srv-pod
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.152.183.98
IPs:               10.152.183.98
Port:              <unset>  8080/TCP
TargetPort:        80/TCP
Endpoints:         10.1.161.239:80,10.1.174.25:80,10.1.174.26:80
Session Affinity:  None
Events:            <none>
```

**4) Check if on scaling the deployment, new pods automatically added to the service**

- _Scale the deployment using following command_
``` sh
kubectl scale deployment nginx-deployment --replicas=10

# Output
deployment.apps/nginx-deployment scaled
```

- _Check if new pods are created_
``` sh
kubectl get pods --show-labels

# output
NAME                                READY   STATUS    RESTARTS   AGE     LABELS
nginx-deployment-5d48b7bd6c-ntnx4   1/1     Running   0          9m40s   app=nginx-srv-pod,pod-template-hash=5d48b7bd6c
nginx-deployment-5d48b7bd6c-b9cxh   1/1     Running   0          9m39s   app=nginx-srv-pod,pod-template-hash=5d48b7bd6c
nginx-deployment-5d48b7bd6c-pvr5f   1/1     Running   0          9m39s   app=nginx-srv-pod,pod-template-hash=5d48b7bd6c
nginx-deployment-5d48b7bd6c-86cfn   1/1     Running   0          24s     app=nginx-srv-pod,pod-template-hash=5d48b7bd6c
nginx-deployment-5d48b7bd6c-dsjzx   1/1     Running   0          23s     app=nginx-srv-pod,pod-template-hash=5d48b7bd6c
nginx-deployment-5d48b7bd6c-6qtdv   1/1     Running   0          23s     app=nginx-srv-pod,pod-template-hash=5d48b7bd6c
nginx-deployment-5d48b7bd6c-sjw4w   1/1     Running   0          24s     app=nginx-srv-pod,pod-template-hash=5d48b7bd6c
nginx-deployment-5d48b7bd6c-674rz   1/1     Running   0          24s     app=nginx-srv-pod,pod-template-hash=5d48b7bd6c
nginx-deployment-5d48b7bd6c-dfz5l   1/1     Running   0          23s     app=nginx-srv-pod,pod-template-hash=5d48b7bd6c
nginx-deployment-5d48b7bd6c-xc76k   1/1     Running   0          23s     app=nginx-srv-pod,pod-template-hash=5d48b7bd6c
```

- _Check if IP's of new pods added to the service_
``` sh
kubectl describe service nginx-service

# output
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx-srv-pod
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.152.183.98
IPs:               10.152.183.98
Port:              <unset>  8080/TCP
TargetPort:        80/TCP
Endpoints:         10.1.161.235:80,10.1.161.238:80,10.1.161.239:80 + 7 more...
Session Affinity:  None
Events:            <none>
```

- _Check details of the endpoints_
``` sh
kubectl describe endpoints nginx-service

# output
Name:         nginx-service
Namespace:    default
Labels:       <none>
Annotations:  <none>
Subsets:
  Addresses:          10.1.161.235,10.1.161.238,10.1.161.239,10.1.161.240,10.1.161.241,10.1.174.25,10.1.174.26,10.1.174.29,10.1.174.30,10.1.174.31
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  80    TCP

Events:  <none>
```

**5) Cleanup**
``` sh
kctl delete -f nginx-service-all.yaml

# output
deployment.apps "nginx-deployment" deleted
service "nginx-service" deleted
```

### Type of Service 

![[Pasted image 20230506184952.png]]
#### 1) ClusterIP 
- This is the default type of service in Kubernetes. 
- It creates a service which can be accessed by other applications in the kubernetes cluster, without allowing external access.

_Service example we seen is example of ClusterIP_

**Use Cases**
- Inter service communication within the cluster. For example, communication between the front-end and back-end components of your app.

#### 2) NodePort
-   NodePort service is an extension of ClusterIP service. 
- It exposes the service outside of the cluster by adding a cluster-wide static port on each node. Each node proxies that port into your Service.
-   You can contact the NodePort Service, from outside the cluster, by requesting \<Node IP\>:\<Node Port\>
-   Node port must be in the range of 30000–32767. Manually allocating a port to the service is optional. If it is undefined, Kubernetes will automatically assign one.
-   If you are going to choose node port explicitly, ensure that the port was not already used by another service.

**Use Cases**

- When you want to enable external connectivity to your service.
-  Using a NodePort gives you the freedom to set up your own load balancing solution, to configure environments that are not fully supported by Kubernetes, or even to expose one or more nodes’ IP's directly.
-  Prefer to place a load balancer above your nodes to avoid node failure.

**1) Create service using yaml file - nginx-npservice.yaml**
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
   name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-srv-pod
  template:
    metadata:
      labels:
        app: nginx-srv-pod
    spec:
      containers:
      - name: nginx
        image: nginx:stable
        ports:
        - containerPort: 80 
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-npservice
spec:
  type: NodePort
  selector:
    app: nginx-srv-pod
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```

**2) Create service using CLI**

- _Create deployment_
``` sh
kubectl create deployment nginx-deployment --image=nginx:latest --replicas=3 --port=80 --dry-run=client -o yaml

# Output
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
status: {}
```
_If output is as expected, re-run the command without --dry-run and -o arguments._

- _Create service_
``` sh
kubectl expose deployment nginx-deployment --name=nginx-npservice --port=8080 --target-port=80 --type=NodePort --dry-run=client -o yaml

# Output

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx-deployment
  name: nginx-npservice
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx-deployment
  type: NodePort
status:
  loadBalancer: {}
```
_If output is as expected, re-run the command without --dry-run and -o arguments._

#### 3) LoadBalancer

- LoadBalancer service is an extension of NodePort service. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
-   By default kubernetes didn't provide load balancer, it mostly depends on cloud providers (like AWS, GCP.. etc.) to provide there own implementation.
- If load balance implementation not provided, this type of service behaves as NodePort service.
- On local kubernetes installations, we need to install and configure LB. Generally  on private/local installations, metallb is used as the loadbalancer. Please check section _Installation = microk8s_ to configure it.

**Use cases**
- When you are using a cloud provider to host your Kubernetes cluster.

**1) Create service using yaml file - nginx-lbservice.yaml**
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
   name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-srv-pod
  template:
    metadata:
      labels:
        app: nginx-srv-pod
    spec:
      containers:
      - name: nginx
        image: nginx:stable
        ports:
        - containerPort: 80 
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-lbservice
spec:
  type: LoadBalancer
  selector:
    app: nginx-srv-pod
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```

**2) Create service using CLI**
- Follow the same steps defined for NodePort., just while creating service change type to _--type=LoadBalancer_.
