
## Airgap Installation process

1) Download Required Image/Images
``` sh
curl -L -o k3s-airgap-images-amd64.tar.zst "https://github.com/k3s-io/k3s/releases/download/v1.35.2%2Bk3s1/k3s-airgap-images-amd64.tar.zst"
```

2) Copy tar file to "/var/lib/rancher/k3s/agent/images/"

```  sh
sudo mkdir -p /var/lib/rancher/k3s/agent/images/
sudo cp k3s-airgap-images-amd64.tar.zst /var/lib/rancher/k3s/agent/images/k3s-airgap-images-amd64.tar.zst
```

3) Create cache file - Need for using local images
``` sh
touch /var/lib/rancher/k3s/agent/images/.cache.json
```

4) Download k3s binary
``` sh
sudo curl -Lo /usr/local/bin/k3s https://github.com/k3s-io/k3s/releases/download/v1.35.2%2Bk3s1/k3s

sudo chmod +x /usr/local/bin/k3s
```

5) Download install script for airgap installation

``` sh
curl -Lo install.sh https://get.k3s.io
chmod +x install.sh
```

6) Execute install

``` sh
INSTALL_K3S_SKIP_DOWNLOAD=true ./install.sh
```


## Using local images

1) Save image as tar file
``` sh
podman image ls
podman save 9cc24f05f309 >> rockylinux93.tar
```

2) Copy tar file to "/var/lib/rancher/k3s/agent/images/"
3) In pod def, always set "imagePullPolicy: IfNotPresent"
``` yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: rockypod
  name: rockypod
spec:
  containers:
  - image: rockylinux:9.3
    imagePullPolicy: IfNotPresent 
    name: rockypod
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from K3s! && sleep 3600"]
```

4) We can confirm image name using -
``` sh
k3s ctr image list
```

## Upgrade k3s installation

1) Download image as defined in step-1 of "Airgap Installation process" and run "INSTALL_K3S_SKIP_DOWNLOAD=true ./install.sh" again.
2) To upgrade k3s utilities use step-4 of "Airgap Installation process"


## Uninstallation

```
sudo /usr/local/bin/k3s-uninstall.sh
```
