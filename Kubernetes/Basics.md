

## [[Installation - kubeadm]]

OR

## Using minikube

* Start and stop cluster
```bash
minikue start
minikube stop
```

## Deployment

	Pod is a instance of container.
	A deployment can have any number of pods to get job done.

* Deployment File ("deployment.yaml")

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomee-deployment
spec:
  selector:
    matchLabels:
      app: tomee
  replicas: 1
  template:
    metadata:
      labels:
        app: tomee
    spec:
      containers:
      - name: tomee
        image: tomee:latest
        ports:
        - containerPort: 8080
```

* Command to deploy
```bash
kubectl apply -f ./deployment.yaml
```

* Exposing service for external access

```bash        
kubectl expose deployment tomee-deployment --type NodePort
```
* List down pods

```bash
kubectl get pods
```

* List service URLS
```bash
minikube service tomee-deployment --url
```

* Check web page
```bash
curl http://192.168.49.2:31043
```




