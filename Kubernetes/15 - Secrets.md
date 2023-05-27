A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) specification or in a [container image](https://kubernetes.io/docs/reference/glossary/?all=true#term-image). Using a Secret means that you don't need to include confidential data in your application code.


### Create Secrets

#### 1) Using literal -

- Encode password "dbpassword123" value using Base64 encode
``` sh
echo "dbpassword123" | base64

# Output
ZGJwYXNzd29yZDEyMwo=
```

- Create secret
``` yaml
kubectl create secret generic db-pwd-01 --from-literal=pwd=dbpassword123
```

- Check secret
``` yaml
kubectl get secret db-pwd-01 -o yaml

# Output
apiVersion: v1
data:
  pwd: ZGJwYXNzd29yZDEyMwo=
kind: Secret
metadata:
  creationTimestamp: "2023-05-27T15:34:44Z"
  name: db-pwd-01
  namespace: default
  resourceVersion: "73435"
  uid: da58ac87-8286-4676-8f80-6e870eafb49c
type: Opaque
```

- Decode secret
``` sh
echo "ZGJwYXNzd29yZDEyMwo=" | base64 -d

# Output
dbpassword123
```

#### 2) Using file -

- Create file "pwdFile.txt"
``` Text
dbpasspord123
```

- Create secret
``` sh
kubectl create secret generic db-pwd-01 --from-file=pwd=./pwdFile.txt
```

- Check secret
``` sh
kubectl get secret db-pwd-01 -o yaml

# Output
apiVersion: v1
data:
  pwd: ZGJwYXNzcG9yZDEyMwo=
kind: Secret
metadata:
  creationTimestamp: "2023-05-27T15:46:06Z"
  name: db-pwd-01
  namespace: default
  resourceVersion: "74822"
  uid: bc0f48d3-7ba5-4dd2-af81-e7e5f32b7e59
type: Opaque
```

### Use secret in the pod

``` yaml
apiVersion: v1 
kind: Pod 
metadata: 
   name: pod-sc-use01
spec: 
  containers: 
    - image: busybox:latest
      name: busybox-01
      command: ['sh', '-c', 'while true; do sleep 3600; done']
      volumeMounts: 
        - name: pwd-secret 
          mountPath: /var/local/dbpwd 
          readOnly: true 
  volumes: 
    - name: pwd-secret
      secret: 
        secretName: db-pwd-01
```