
### Delete all the resources in specific namespace

1) List namespace
``` sh
kubectl get ns
```

2) Delete all the resource in selected namespace
``` sh
kubectl delete all --all -n ingress-nginx
```

3) Connect with specific container in multi-container pod
``` sh
kubectl exec pod-name -c container-name -it -- sh
```

4) Get details of specific API
``` sh
kubectl api-resources | grep configmap
```
