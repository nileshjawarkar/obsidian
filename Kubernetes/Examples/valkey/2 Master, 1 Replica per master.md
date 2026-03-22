
### Step 1 - Create valkey image with DNS tools
Here we required dns tools in valkey image, so we need to create our own image -

``` Dockerfile
# Use official Valkey image as base
FROM valkey/valkey:9

# Install dig (dnsutils) on top of Valkey image
USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends dnsutils && \
    rm -rf /var/lib/apt/lists/*

# Switch back to default user if needed
USER valkey

# Working directory
WORKDIR /data

# Expose Valkey ports
EXPOSE 6379 16379

# Default command: run Valkey in cluster mode
CMD ["valkey-server", "--cluster-enabled", "yes", "--cluster-config-file", "/data/nodes.conf", "--appendonly", "yes"]
```

Build image using command - 
``` sh
docker build . -t valkey-n-dnsutils:9
```

Here we expect Dockerfile in same director. If you place it some where, use cd $DIR .

### Step 2 : Create manifest for valkey statefulset, its configmap and headless service

valkey-srv.yaml
``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: valkey-cluster-config
data:
  valkey.conf: |
    port 6379
    cluster-enabled yes
    cluster-config-file /data/nodes.conf
    cluster-node-timeout 5000
    appendonly yes
    dir /data
    logfile ""
    loglevel notice
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: valkey-ss
spec:
  replicas: 4   # 2 masters + 2 replicas
  selector:
    matchLabels:
      app: valkey-pod
  template:
    metadata:
      labels:
        app: valkey-pod
    spec:
      containers:
        - name: valkey
          image: valkey-n-dnsutils:9
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 6379
          command: ["valkey-server", "/etc/valkey/valkey.conf"]
          volumeMounts:
            - name: config
              mountPath: /etc/valkey
            - name: data
              mountPath: /data
          livenessProbe:
            exec:
              command: ["valkey-cli", "ping"]
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 10
            successThreshold: 1
            failureThreshold: 4

          readinessProbe:
            exec:
              command: ["valkey-cli", "ping"]
            initialDelaySeconds: 15
            periodSeconds: 10
            timeoutSeconds: 10
            successThreshold: 1
            failureThreshold: 4

      volumes:
        - name: config
          configMap:
            name: valkey-cluster-config

  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 200Mi
  persistentVolumeClaimRetentionPolicy:
      whenDeleted: Delete
      whenScaled: Delete
---
apiVersion: v1
kind: Service
metadata:
  name: valkey-cluster-svch
spec:
  ports:
    - port: 6379
      name: valkey
  clusterIP: None
  selector:
    app: valkey-pod

```

### Step 3: Create manifest for client app, which will also configured valkey for cluster mode.

valkey-client-srv.yaml
``` yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: valkey-client-ss
spec:
  replicas: 2
  selector:
    matchLabels:
      app: valkey-client-pod
  template:
    metadata:
      labels:
        app: valkey-client-pod
    spec:
      initContainers:
        - name: init-valkey
          image: valkey-n-dnsutils:9
          imagePullPolicy: IfNotPresent
          command: 
            - sh
            - -c
            - |
              echo "[VALKEY] Configure valkey cluster ..."
              PRIMARY_NODE=""
              CLUSTER_NODES=""

              RETRIES=1000
              while [ $RETRIES -gt 0 ]
              do
                  PODS=$(nslookup valkey-cluster-svch | awk '/^Address: / { print $2 }')
                  COUNT=0
                  CLUSTER_NODES=""
                  for ip in $PODS; do
                      CLUSTER_NODES="$CLUSTER_NODES $ip:6379"
                      PRIMARY_NODE=$ip
                      COUNT=$((COUNT + 1))
                  done
                  if [ "$COUNT" -eq 4 ]; then
                      break
                  fi
                  RETRIES=$((RETRIES - 1))
                  sleep 2
              done

              if [ -n "$PRIMARY_NODE" ]; then
                  OUTPUT=$(valkey-cli -h "$PRIMARY_NODE" -p 6379 cluster info 2>&1)
                  if echo "$OUTPUT" | grep -q "cluster_state:ok"; then
                      echo "[VALKEY] Cluster is already configured"
                      exit 0
                  else
                      echo "[VALKEY] $PRIMARY_NODE debug=$OUTPUT"
                      CLUSTER_NODES=$(echo "$CLUSTER_NODES" | xargs)  # trims extra spaces
                      echo "[VALKEY - Configuring nodes for cluster mode] $CLUSTER_NODES"
                      valkey-cli --cluster create $CLUSTER_NODES --cluster-replicas 1 --cluster-yes
                      exit $?
                  fi
              fi
              exit 1
      containers:
      - image: valkey-client-app:1
        imagePullPolicy: IfNotPresent
        name: client-app
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: valkey-client-svc
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 31080

  selector:
    app: valkey-client-pod

```

### Step 4 - We used here 1 tomee web app image.

Here we build local maven app, which create war and then this war is package in image.

``` Dockerfile
FROM tomee:plume

WORKDIR /tmp

# Copy WAR file into /tmp
COPY valkey_webapp/target/valkey_webapp-0.0.1.war /tmp/valkey_webapp.war

# Install unzip (if not already present) and extract WAR
RUN apt-get update && apt-get install -y unzip \
    && mkdir /tmp/valkey_webapp \
    && unzip /tmp/valkey_webapp.war -d /tmp/valkey_webapp \
    && rm -rf /usr/local/tomee/webapps/ROOT \
    && cp -r /tmp/valkey_webapp /usr/local/tomee/webapps/ROOT \
    && rm -rf /var/lib/apt/lists/* /tmp/valkey_webapp.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
```

We use following command to build this image. Again we are considering that dockerfile is in current directory.

``` sh
docker build . -t valkey-client-app:1
```

### Step 4: deployment

``` sh
kubectl apply -f valkey-srv.yaml
kubectl apply -f valkey-client-srv.yaml
```