Kubernetes uses following objects. In this section will see what are these objects and description of them
- Pod
- ReplicaSet
- Deployment
- Service
- Volumn
- Name space
- ConfigMap and Secret
- Stateful Set
- Daemon Set

## Understanding Kubernetes objects[](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#kubernetes-objects)

- _Kubernetes objects_ are persistent entities in the Kubernetes system. 
- Kubernetes uses these entities to represent the state of your cluster. 
- A Kubernetes object is a "record of intent"--once you create the object, the Kubernetes system will constantly work to ensure that object exists. 
- By creating an object, you're effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's _desired state_.

To work with Kubernetes objects--whether to create, modify, or delete them--you'll need to use the [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/). When you use the `kubectl` command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you.

### Describing a Kubernetes object[](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#describing-a-kubernetes-object)

- Kubernetes uses yaml to define the objects. 
- When you create an object in Kubernetes, you must provide the object spec that describes its desired state, as well as some basic information about the object.
- For creating the objects,  you'll need to set values for the following fields:
	-   `apiVersion` - Which version of the Kubernetes API you're using to create this object
	-   `kind` - What kind of object you want to create
	-   `metadata` - Data that helps uniquely identify the object, including a `name` string, `UID`, and optional `namespace`
	-   `spec` - What state you desire for the object. The precise format of the object `spec` is different for every Kubernetes object, and contains nested fields specific to that object.


### Pods

- Pods are the smallest deployable units of computing that you can create and manage in Kubernetes. 
- A Pod is a group of one or more [containers](https://kubernetes.io/docs/concepts/containers/), with shared storage and network resources, and a specification for how to run the containers. 
- Each Pod will have a unique IP Address in a particular cluster, it allows the usage of ports without any conflicts.

_nginx-pod.yaml - Simple Pod Configuration using yaml_
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

- Above yaml file show properties required to define a Pod.

#### 1) Creation, listing and deletion of pods
- Creation - `kubctl apply -f nginx-pod.yaml`  
- List
	- `kubectl get pods -o wide` - List pods in default namespace
	- `kubectl get pods --all-namespaces -o wide` - List pods in all namespaces.
- Deletion - 
	- `kubectl delete -f nginx-pod.yaml`
	- `kubectl delete pod nginx-pod`

#### 2) Pod info
- Describe pod - `kubectl describe pod nginx-pod`

#### 3) Generate pod yaml using CLI
- `kubectl run nginx-pod --image=nginx:latest --port=80 --dry-run=client -o yaml`
_output_
``` yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-pod
  name: nginx-pod
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
- Now we can use above output and modify it as per the requirements

#### 4) Running a command right after container start - This will be very helpful for testing.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-pod
  name: nginx-pod
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
    lifecycle:
      postStart:
        exec: 
           command: ["/bin/sh", "-c", 'echo "<html><body>NGINX Server, HostName - ${HOSTNAME}</body></html>" > /usr/share/nginx/html/index.html']
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

#### 5) Labels and selectors

- Labels are nothing but key & value pair associated with kubernetes objects
- And these labels can be used as selectors to filter pods.

**Next we will see how we can use them in real**

_1) Lets create 2 pods.._
- We can also use yaml file we created earlier and modify it to have labels. Please check generated yaml file, it has labels attribute in metadata section.
- Here to keep it simple, I am using `kubectl run`
- `kubectl run pod-01 --image nginx:latest`
- `kubectl run pod-02 --image nginx:latest`

_2) Check if they have any labels_
- `kubectl get pods -o wide --show-labels`
``` Text
NAME     READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES   LABELS
pod-01   1/1     Running   0          29s   10.1.174.8   ubuntusrv02   <none>           <none>            run=pod-01
pod-02   1/1     Running   0          25s   10.1.174.9   ubuntusrv02   <none>           <none>            run=pod-02
```

_3) Add new labels to each pod_
- `kubectl label pod pod-01 env=prod`
- `kubectl label pod pod-02 env=dev`

_4) Using labels as a selector to filter out pods_
- `kubectl get pods -l env=dev` - This command will list only those pods which has label 'dev'
``` Text
NAME     READY   STATUS    RESTARTS   AGE
pod-02   1/1     Running   0          2m37s
```

_5) Remove label_
- `kubectl label pod pod-01 env-` - At end of label _-_ sign indicate that label should be removed.
- After executing above command , please run `kubectl get pods -o wide --show-labels`
``` Text
NAME     READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES   LABELS
pod-02   1/1     Running   0          10m   10.1.174.9   ubuntusrv02   <none>           <none>            env=dev,run=pod-02
pod-01   1/1     Running   0          10m   10.1.174.8   ubuntusrv02   <none>           <none>            run=pod-01
```
- We can see that label _env_ is removed from pod-01


