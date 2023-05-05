
Service provides single IP address and domain (gateway) for multiple replicas of the pod (endpoints) and serve as a load balancer and help pod's to scale.

![[Pasted image 20230504223729.png]]

### Demo - Service concepts:

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

Test connection between frontend and endpoints
- 
**3) Create service**

**4) Add endpoint to the service**


#### Create service using yaml

**In work**

### Type of Service 
1) ClusterIP 
	- This is the default type of service in Kubernetes. 
	- It creates a service which can be accessed by other applications in the kubernetes cluster, without allowing external access.
2) NodePort
3) LoadBalancer