``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx-hpa
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-dep
  name: nginx-dep
  namespace: nginx-hpa
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
  namespace: nginx-hpa
spec:
  ports:
  - port: 8081
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx-dep
---
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa1
  namespace: nginx-hpa
spec:
  maxReplicas: 4
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-dep
  targetCPUUtilizationPercentage: 30
status:
  currentReplicas: 0
  desiredReplicas: 
```


#### CLI

``` sh
kubectl autoscale deployment nginx-dep --cpu-percent=30 --min=2 --max=4 --name=nginx-hpa-1 --namespace=nginx-hpa  --dry-run=client -o yaml

# output

apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: nginx-hpa-2
spec:
  maxReplicas: 4
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-dep
  targetCPUUtilizationPercentage: 30
status:
  currentReplicas: 0
  desiredReplicas: 0
```

- _Note - Above CLI command will not add namespace to metadata. Need to be added manually._

