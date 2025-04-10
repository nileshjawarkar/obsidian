- A ReplicaSet's purpose is to maintain a required number of replica's of specific Pod, running at any given time. 
- As such, it is often used to guarantee the availability of a specified number of identical Pods.

#### Current state VS Desired state

- Desired state - Required number of replica's that must be running.
- Current state - Actual number of replica's currently running.

![repsetimg](../../../../images/img20230503135324.png)


- As show in above snap, when any pod is down which result is current state (2) which is not a desired state (3), then ReplicaSet will start new pod in order to maintain desire state.

#### Let's create out 1st replicaset 

_1) nginx-replicaset.yaml_
``` yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginxrs
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
```
- Point to note hear is _template section_ of replicaset is _pod definition_. In this case of pod definition, we don't need to specify _apiVersion_ & _kind_.
- ReplicaSet uses label to identify pod belong to the replicaset.
- ReplicaSet uses selector matchLabels _tier: nginx-rs-pod_ and same label need to be defined on the pod.

_2) create replicaset_
- `kubectl apply -f nginx-replicaset.yaml`

_3) List replicaset and its pods_
- `kubectl get replicaset`
``` Text
NAME      DESIRED   CURRENT   READY   AGE
nginxrs   3         3         3       3m23s
```

- `kubectl get pods --show-labels`
``` Text
NAME            READY   STATUS    RESTARTS   AGE     LABELS
nginxrs-n6jvx   1/1     Running   0          6m34s   tier=nginx-rs-pod
nginxrs-nx4zr   1/1     Running   0          6m34s   tier=nginx-rs-pod
nginxrs-jtmzh   1/1     Running   0          6m34s   tier=nginx-rs-pod
```

_4) In this example of replicaset, we defined desired state as 3. Lets kill one of the pod to test desire state promise(it will try to keep running number of pod defined in desire state) made by replicaset._

- `kubectl delete pod nginxrs-n6jvx` - Kill the pod
- `kubectl get pods --show-lables` - This command will show the new pod started for replicaset
``` Text
NAME            READY   STATUS    RESTARTS   AGE   LABELS
nginxrs-nx4zr   1/1     Running   0          17m   tier=nginx-rs-pod
nginxrs-jtmzh   1/1     Running   0          17m   tier=nginx-rs-pod
nginxrs-sz57n   1/1     Running   0          9s    tier=nginx-rs-pod
```

_5) Delete the replicaset_
- `kubectl delete -f nginx-replicaset.yaml`
- `kubectl delete replicaset nginxrs`
