_IN-WORK_

``` yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app: nginx-ss
  name: nginx-ss
spec:
  serviceName: nginx-hsvc
  replicas: 2
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
          - containerPort: 80
        volumeMounts:
          - name: www
            mountPath: "/usr/share/nginx/html"
  volumeClaimTemplates:
     - metadata:
         name: www
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: mayastor
         resources:
           requests:
             storage: 1Gi
```
