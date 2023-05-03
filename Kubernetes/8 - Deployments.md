Deployment provide replication facility using ReplicaSet, along with many other important features, such as rollout, rollback etc.

#### Create first deployment

_1) Create deployment using nginx-deployment.yaml_
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: nginx-rs-pod
  template:
    metadata:
      labels:
        tier: nginx-rs-pod
    spec:
      containers:
      - name: nginx
        image: nginx:stable
        ports:
        - containerPort: 80        
```

- `kubectl apply -f nginx-deployment.yaml` - Create deployment

_2) List deployment_
- `kubectl get deployment`
``` Text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           3m38s
```

_3) List replicaset_
- `kubectl get replicaset`
``` Text
NAME                         DESIRED   CURRENT   READY   AGE
nginx-deployment-59f966b54   3         3         0       14s
```

_4) List pods_
- `kubectl get pods --show-labels `
``` Text
NAME                               READY   STATUS    RESTARTS   AGE   LABELS
nginx-deployment-59f966b54-dl4r8   1/1     Running   0          47s   pod-template-hash=59f966b54,tier=nginx-rs-pod
nginx-deployment-59f966b54-l7db6   1/1     Running   0          47s   pod-template-hash=59f966b54,tier=nginx-rs-pod
nginx-deployment-59f966b54-bmfj8   1/1     Running   0          47s   pod-template-hash=59f966b54,tier=nginx-rs-pod
```

- `kubectl get pods -l tier=nginx-rs-pod `
``` Text
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-59f966b54-dl4r8   1/1     Running   0          7m30s
nginx-deployment-59f966b54-l7db6   1/1     Running   0          7m30s
nginx-deployment-59f966b54-bmfj8   1/1     Running   0          7m30s
```

#### Rollout 

In-work

#### Rollback

In-work
