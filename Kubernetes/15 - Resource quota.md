
``` yaml
piVersion: v1
kind: Namespace
metadata:
  name: res-quota
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: nginx-dep-quota
  namespace: res-quota
spec:
  hard:
    limits.cpu: 600m
    limits.memory: 600Mi 
    requests.cpu: 300m
    requests.memory: 300Mi 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-dep
  name: nginx-dep
  namespace: res-quota
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-dep
  template:
    metadata:
      labels:
        app: nginx-dep
    spec:
      volumes:
        - name: data
          emptyDir:
            sizeLimit: 300Mi
      initContainers:
        - name: rocky-init
          image: rocky:9
          command:
            - bash
            - "-c"
            - echo "This is from $HOSTNAME" > /usr/share/nginx/html/index.html
          volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: data
          resources:
            limits:
              cpu: 200m
              memory: 200Mi
            requests:
              cpu: 100m
              memory: 100Mi
      containers:
        - image: nginx:latest
          name: nginx
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: data
          resources:
            limits:
              cpu: 200m
              memory: 200Mi
            requests:
              cpu: 100m
              memory: 100Mi
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-svc
  name: nginx-svc
  namespace: res-quota
spec:
  ports:
  - port: 8081
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx-dep
```