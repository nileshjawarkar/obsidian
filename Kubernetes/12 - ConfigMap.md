Many applications rely on configuration which is used during either application initialization or runtime. Most times, there is a requirement to adjust values assigned to configuration parameters. ConfigMaps are a Kubernetes mechanism that let you inject configuration data into application [pods](https://kubernetes.io/docs/concepts/workloads/pods/).


__Download 2 properties files to the mapData directory__
``` sh
# Download the sample files into `configure-pod-container/configmap/` directory
wget https://kubernetes.io/examples/configmap/game.properties -O ./game.properties
wget https://kubernetes.io/examples/configmap/ui.properties -O ./ui.properties
```

## Create config map using CLI

#### 1) Using file - 
``` sh
kubectl create configmap cf01 --from-file=./mapData/game.properties --dry-run=client -o yaml

# Output
apiVersion: v1
data:
  game.properties: |-
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cf01
```

#### 2) Using multiple files - 
``` sh
kubectl create configmap cf01 --from-file=./mapData/game.properties --from-file=./mapData/ui.properties --dry-run=client -o yaml

# Output

apiVersion: v1
data:
  game.properties: |-
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
  ui.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
    how.nice.to.look=fairlyNice
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cf01
```


#### 3) Using literal -
``` sh
kubectl create configmap cf01 --from-literal=special.how=very --from-literal=special.type=charm --dry-run=client -o yaml

# Output
apiVersion: v1
data:
  special.how: very
  special.type: charm
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cf01
```


### Define container environment variables using ConfigMap data

1) Create config map
``` sh
kubectl create configmap cf01 --from-literal=special.how=very --from-literal=special.type=charm
```

2) Create pod to use it - import specific key
``` yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-cm-use01
  name: pod-cm-use01
spec:
  containers:
  - image: busybox:latest
    name: busybox-01
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    env:
       - name: SPECIAL_LEVEL_KEY
         valueFrom:
           configMapKeyRef:
              name: cf01
              key: special.how
```


OR

3)  Create pod to use it - import config map as env
``` yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-cm-use02
  name: pod-cm-use02
spec:
  containers:
  - image: busybox:latest
    name: busybox-01
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    envFrom:
       - configMapRef:
          name: cf01
```


### Add ConfigMap data to a Volume

1) Use ConfigMap created in previous topic.
2) Create POD to use configmap as volume
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-use03
spec:
  containers:
  - image: busybox:latest
    name: busybox-01
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
     - name: config-volume
       configMap:
          name: cf01
```