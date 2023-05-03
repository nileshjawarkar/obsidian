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

#### Rollout - Updating the deployment

We all know application keeps changing for various reasons, it may due to bugs, performance improvements etc. Rollout is about the deploying the updated application to use.

In kubernetes case, application update will always lead to image update.
Now for study purpose, lets consider in our deployment image changed from _stable_ to _latest_ .
_1) Perform update_
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
        image: nginx:latest
        ports:
        - containerPort: 80        
```

- `kubectl apply -f nginx-deployment.yaml`
``` Text
deployment.apps/nginx-deployment configured
```

OR update image directly

- `kubectl set image deployment/nginx-deployment nginx=httpd:latest`
- `nginx=<New Image:Version>` - What is nginx here? Its a name of the container used in deployment.
- `--record` -  Above command can use  record option, to record the command used for the upgrade and will be display in the rollout history. But look like it is deprecated now. 

_2) Check replicaset_
- `kubectl get replicaset` - We can see 2 replicaset now. After some time older replicaset will have zero value for desire state, current and ready. It indicate rollout is complete.
``` Text
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-59f966b54    2         2         2       15m
nginx-deployment-589c985f58   2         2         1       21s
```

_3) Check pods_
- `kubectl get pods -l tier=nginx-rs-pod` 
``` Text
NAME                                READY   STATUS        RESTARTS   AGE
nginx-deployment-59f966b54-dl4r8    1/1     Running       0          15m
nginx-deployment-589c985f58-sxlhr   1/1     Running       0          30s
nginx-deployment-589c985f58-9vpvf   1/1     Running       0          27s
nginx-deployment-59f966b54-bmfj8    1/1     Terminating   0          15m
nginx-deployment-589c985f58-rtcjw   0/1     Pending       0          2s
```
- We can see here that pods related to older version are terminating and pods with new version are starting.
- After some we can see pods related to updated version only.
``` Text
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-589c985f58-sxlhr   1/1     Running   0          47m
nginx-deployment-589c985f58-9vpvf   1/1     Running   0          47m
nginx-deployment-589c985f58-rtcjw   1/1     Running   0          46m
```

_4) Upgrade strategy (RollingUpdateStrategy)_
- `kubectl describe deployment nginx-deployment` 
``` Text
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Wed, 03 May 2023 09:39:19 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               tier=nginx-rs-pod
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  tier=nginx-rs-pod
  Containers:
   nginx:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-589c985f58 (3/3 replicas created)
Events:          <none>
```
- We can see here `RollingUpdateStrategy:  25% max unavailable, 25% max surge` strategy is used for the upgrade.
- [Please check this article for more details - A deep dive into Kubernetes Deployment strategies](https://www.educative.io/blog/kubernetes-deployments-strategies)

_5) max surge and max unavailable_
![[Pasted image 20230503164439.png]]

#### Rollback

If for some reason, latest updated version is not working as expected and we need to revert back to older version. This is what rollback is, reverting back to older version.

_1) Check rollout history_
- `kubectl rollout history deployment.v1.apps/nginx-deployment` OR
- `kubectl rollout history deployment/nginx-deployment` OR
- `kubectl rollout history deployment nginx-deployment` 
``` Text
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

_2) Check changes between the revision_
- `kubectl rollout history deployment.v1.apps/nginx-deployment --revision 1`
``` Text
deployment.apps/nginx-deployment with revision #1
Pod Template:
  Labels:	pod-template-hash=59f966b54
	tier=nginx-rs-pod
  Containers:
   nginx:
    Image:	nginx:stable
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

- `kubectl rollout history deployment.v1.apps/nginx-deployment --revision 2`
``` Text
deployment.apps/nginx-deployment with revision #2
Pod Template:
  Labels:	pod-template-hash=589c985f58
	tier=nginx-rs-pod
  Containers:
   nginx:
    Image:	nginx:latest
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

_3) Rollback to revision 1_
- `kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision 1`

#### Create deployment yaml using CLI

- `kubectl create deployment nginx-deployment --image=nginx:latest --replicas=3 --dry-run=client -o yaml`
``` yaml
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
        resources: {}
status: {}
```
