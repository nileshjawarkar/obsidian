- A StatefulSet runs a group of Pods, and maintains a sticky identity for each of those Pods. 
- This is useful for managing applications that need 
	- A stable, unique network identity - Unlike a Deployment, a StatefulSet maintains a sticky identity for each of its Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.
	- Persistent storage - If you want to use storage volumes to provide persistence for your workload, you can use a StatefulSet as part of the solution.
	- Ordered, graceful deployment and scaling.
	- Ordered, automated rolling updates.

### Limitations

- The storage for a given Pod must either be provisioned by a [PersistentVolume Provisioner](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) ([examples here](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/README.md)) based on the requested _storage class_, or pre-provisioned by an admin.
- Deleting and/or scaling a StatefulSet down will _not_ delete the volumes associated with the StatefulSet. This is done to ensure data safety, which is generally more valuable than an automatic purge of all related StatefulSet resources.
- StatefulSets currently require a [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) to be responsible for the network identity of the Pods. You are responsible for creating this Service.
- StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.
- When using [Rolling Updates](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#rolling-updates) with the default [Pod Management Policy](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies) (`OrderedReady`), it's possible to get into a broken state that requires [manual intervention to repair](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#forced-rollback).
### Example:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  clusterIP: None         # headless service
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx          # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3             # by default is 1
  minReadySeconds: 10     # by default is 0
  template:
    metadata:
      labels:
        app: nginx        # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.24
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```
