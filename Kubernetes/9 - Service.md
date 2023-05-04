
Service provides single IP address and domain for multiple replicas of the pod and serve as a load balancer and help pod's to scale.

![[Pasted image 20230504223729.png]]

Type of Service 
1) ClusterIP 
	- This is the default type of service in Kubernetes. 
	- It creates a service which can be accessed by other applications in the kubernetes cluster, without allowing external access.
2) NodePort
3) LoadBalancer