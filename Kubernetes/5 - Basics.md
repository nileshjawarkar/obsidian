
## Understanding Kubernetes objects[](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#kubernetes-objects)

_Kubernetes objects_ are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. A Kubernetes object is a "record of intent"--once you create the object, the Kubernetes system will constantly work to ensure that object exists. By creating an object, you're effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's _desired state_.

To work with Kubernetes objects--whether to create, modify, or delete them--you'll need to use the [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/). When you use the `kubectl` command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you.

### Describing a Kubernetes object[](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#describing-a-kubernetes-object)

When you create an object in Kubernetes, you must provide the object spec that describes its desired state, as well as some basic information about the object (such as a name).

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

In the `.yaml` file for the Kubernetes object you want to create, you'll need to set values for the following fields:

-   `apiVersion` - Which version of the Kubernetes API you're using to create this object
-   `kind` - What kind of object you want to create
-   `metadata` - Data that helps uniquely identify the object, including a `name` string, `UID`, and optional `namespace`
-   `spec` - What state you desire for the object

The precise format of the object `spec` is different for every Kubernetes object, and contains nested fields specific to that object.


## Pods

- _Pods_ are the smallest deployable units of computing that you can create and manage in Kubernetes. 
- A Pod is a group of one or more [containers](https://kubernetes.io/docs/concepts/containers/), with shared storage and network resources, and a specification for how to run the containers. 
- A Pod is similar to a set of containers with shared namespaces and shared filesystem volumes

### ReplicaSet

- A ReplicaSet's purpose is to maintain a required number of replica's of specific Pod, running at any given time. 
- As such, it is often used to guarantee the availability of a specified number of identical Pods.

