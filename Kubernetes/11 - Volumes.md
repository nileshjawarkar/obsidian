Kubernetes volumes provide solution for the following problems -
- As on container restart, it will lose  all of the files that were created or modified during the lifetime of the container.  Kubernetes volume provide a way to persist the data and make it available even after the restart.
- Also multiple container running in the pod, may need to share some files. Kubernetes volume is shared among the container of the same pod's, so the data created by one container can be accessed by another container.


## Type of volumes
- hostPath, emptyDir, nfs, OpenEBS (mayaStor) and many more
- Secret, ConfigMap - as a volume. We will discuss these type of volumes in the sections related to these topics.

 Note - Basically  kubernetes does not provide any volume implementation on it own. It exposes basic infra that can be adopted to implement various types of volume types.

## Persistent Volume and Persistent Volume Claim

**Persistent Volume (PV)** − It’s a piece of network storage that has been provisioned by the administrator. It’s a resource in the cluster which is independent of any individual pod that uses the PV.

**Persistent Volume Claim (PVC)** − The storage requested by Kubernetes for its pods is known as PVC. The user does not need to know the underlying provisioning. The claims must be created in the same namespace where the pod is created.

### 1) hostPath 

This type of volume mounts a file or directory from the host node’s filesystem into your pod.

__Using PV and PVC__ - Use of PV and PVC is generic way of using volumes and applicable to almost all type of volumes.

- Create PV 
``` yaml
kind: PersistentVolume
apiVersion: v1
metadata:
   name: host-path-pv01
   labels:
      type: local
spec:
   capacity: 
      storage: 1Gi 
   accessModes:
      - ReadWriteOnce 
   hostPath:
       path: "/home/data"
```

- Create PVC
``` yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
   name: host-path-pvc01
spec:
   accessModes:
      - ReadWriteOnce
   resources:
      requests:
         storage: 1Gi
```

- Create POD - use PVC created above
``` yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-pvc
  name: pod-pvc
spec:
  containers:
  - image: busybox:latest
    name: busybox-01
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    volumeMounts:
      - mountPath: /var/local/data
        name: mydata
  volumes:
    - name: mydata
      persistentVolumeClaim:
        claimName: host-path-pvc01
```

__Without using PV and PVC__ - (May be) This is specific to hostPath type volume

- Create POD
``` yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox-pod
  name: busybox-pod
spec:
  containers:
  - image: busybox:latest
    name: busybox-01
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    volumeMounts:
      - mountPath: /usr/local/test
        name: mydir
      - mountPath: /usr/local/test/testFile.txt
        name: myfile
  volumes:
    - name: mydir
      hostPath:
        path: /usr/local/test
        type: DirectoryOrCreate
    - name: myfile
      hostPath:
        path: /usr/local/test/testFile.txt
        type: FileOrCreate
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```


### 2) emptyDir

This _ephemeral_ volume is created to share some space among the containers of the pod. 
This volume is created when a Pod is first assigned to a Node. It remains active as long as the Pod is running on that node. The volume is initially empty and the containers in the pod can read and write the files in the emptyDir volume. Once the Pod is removed from the node, the data in the emptyDir is erased.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: busbox-pod
spec:
  containers:
  - image: busybox:latest
    name: busybox-01
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  - image: busybox:latest
    name: busybox-02
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 100Mi
```

The `emptyDir.medium` field controls where `emptyDir` volumes are stored. By default `emptyDir` volumes are stored on whatever medium that backs the node such as disk, SSD, or network storage, depending on your environment. If you set the `emptyDir.medium` field to `"Memory"`, Kubernetes mounts a tmpfs (RAM-backed filesystem) for you instead. While tmpfs is very fast, be aware that unlike disks, tmpfs is cleared on node reboot and any files you write count against your container's memory limit.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: busbox-pod
spec:
  containers:
  - image: busybox:latest
    name: busybox-01
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  - image: busybox:latest
    name: busybox-02
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 100Mi
      medium: Memory
```


### 3) OpenEBS - Mayastor

On installing/enabling mayastor, it create 2 storage classes. So for using mayastor, we dont have to create _Persistent Volume_, we can directly create _Persistent Volume claim_.

- Create PVC - Please note use of storage class
``` yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
   name: mydata-pvc
spec:
   storageClassName: mayastor
   accessModes:
      - ReadWriteOnce
   resources:
      requests:
         storage: 1Gi
```

- Create POD 
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:    
  containers:
  - image: busybox:latest
	name: busybox
	command: ['sh', '-c', 'while true; do sleep 3600; done']
	volumeMounts:
	- mountPath: /usr/local/mydata
	  name: mydata
  volumes:
	- name: mydata
	  persistentVolumeClaim:
		claimName: mydata-pvc
```

## Deployments and Persistent Volume Claim

This is about the limitation of deployment. Deployment didnt provide us a way, where we can not create volume claim as per specified number of replicas and use it for the pod.

### Lets discuss it using following example -

- Save following snippet as _busybox-deployment.yaml_
``` yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
   name: mydata-pvc
spec:
   storageClassName: mayastor
   accessModes:
      - ReadWriteOnce
   resources:
      requests:
         storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: busybox-deployment
  name: busybox-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: busybox-pod
  template:
    metadata:
      labels:
        app: busybox-pod
    spec:
      containers:
      - image: busybox:latest
        name: busybox
        command: ['sh', '-c', 'while true; do sleep 3600; done']
        volumeMounts:
        - mountPath: /usr/local/mydata
          name: mydata
      volumes:
        - name: mydata
          persistentVolumeClaim:
            claimName: mydata-pvc
```

- execute `kubectl apply -f busybox-deployment`

- Point to note - 
	1) We have created only one volume claim and we want to create 2 replicas of pod
	2) Volume claim can be attached to one pod only, so out of 2 pod we asked to create, creation of one pod will fails with error _Unable to attach or mount volumes_. 
	3) As we discuss above, deployment didn't provide any way to create multiple volume claim, which can be attached to the each pod created. In such requirements _StatefulSet_ should be used.
- Now modify yaml and set replicas to 1 and re-apply the yaml. 

