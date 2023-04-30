### 9) Delete image

``` sh
$ virsh list --all
$ virsh shutdown ubuntu22042srv01
$ virsh undefine ubuntu22042srv01
```


### 10) Clone image / Import image

*1: Clone*
``` sh
$ virt-clone --original ubuntu22042srv01 \
  --name ubuntu22042srv02 \
  --mac RANDOM \
  --file /home/nilesh/KVM/images/ubuntu22042srv02.img 
```

*OR Import*
``` sh
virt-install --name ubuntu22042srv02 \
--os-variant ubuntu22.04 \
--ram 3072 \
--disk /home/nilesh/KVM/images/ubuntu22042srv02.img,format=qcow2 \
--graphics vnc,listen=0.0.0.0 \
--noautoconsole \
--hvm \
--mac RANDOM \
--network=default \
--import
```

*2: Change host name*
- Modify /etc/hosts
- Modify /etc/hostname

*3: Change machine Id*
- Modify machine id in  `/etc/machine-id`