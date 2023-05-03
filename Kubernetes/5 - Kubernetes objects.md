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


## Pods

- Pods are the smallest deployable units of computing that you can create and manage in Kubernetes. 
- A Pod is a group of one or more [containers](https://kubernetes.io/docs/concepts/containers/), with shared storage and network resources, and a specification for how to run the containers. 
- Each Pod will have a unique IP Address in a particular cluster, it allows the usage of ports without any conflicts.

*nginx-pod.yaml - Simple Pod Configuration using yaml*
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

_Creation, listing and deletion of pods_
- Creation - `kubctl apply -f nginx-pod.yaml`  
- List - `kubectl get pods`
- Deletion - 
	- `kubectl delete -f nginx-pod.yaml`
	- `kubectl delete pod nginx-pod`

### ReplicaSet

- A ReplicaSet's purpose is to maintain a required number of replica's of specific Pod, running at any given time. 
- As such, it is often used to guarantee the availability of a specified number of identical Pods.

