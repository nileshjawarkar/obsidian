
## Create config files to configure new VM

- user-data.yaml

``` yaml
# user-data
#cloud-config
users:
  - name: myuser
    ssh-authorized-keys:
      - ssh-rsa AAAAB3...your-public-key... user@host
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    groups: sudo
    shell: /bin/bash

package_update: true
packages:
  - qemu-guest-agent

runcmd:
  - echo "Cloud-init config executed!" > /etc/motd
```

- meta-data.yaml

``` yaml
# meta-data
instance-id: my-vm
local-hostname: my-vm
```

## Download cloud image that support cloud-init

``` txt
https://cloud-images.ubuntu.com/releases/
```

## Start VM using --cloud-init argument

``` sh
virt-install \
  --name my-vm \
  --memory 2048 \
  --vcpus 2 \
  --disk path=my-vm-disk.qcow2,format=qcow2 \
  --disk path=focal-server-cloudimg-amd64.img,device=disk,bus=virtio \
  --cloud-init user-data=user-data,meta-data=meta-data \
  --os-type linux \
  --os-variant ubuntu20.04 \
  --network network=default,model=virtio \
  --graphics none \
  --console pty,target_type=serial \
  --extra-args 'console=ttyS0,115200n8 serial'
```

1. **`--disk path=my-vm-disk.qcow2,format=qcow2`**:
    - **Purpose**: This option specifies the primary disk image for the virtual machine.
    - **Parameters**:
        - `path=my-vm-disk.qcow2`: This is the file path to the disk image that will be used as the primary storage for the VM.
        - `format=qcow2`: This specifies the format of the disk image. In this case, it is `qcow2`, which stands for QEMU Copy On Write version 2. It is a widely used disk image format that supports features like snapshots and compression.
    
1. **`--disk path=focal-server-cloudimg-amd64.img,device=disk,bus=virtio`**:
    - **Purpose**: This option specifies an additional disk image for the virtual machine, typically an installation or cloud image.
    - **Parameters**:
        - `path=focal-server-cloudimg-amd64.img`: This is the file path to the disk image that will be used as an additional disk. In this case, it is an Ubuntu cloud image.
        - `device=disk`: This specifies the type of device. Here, it is set as a disk device.
        - `bus=virtio`: This specifies the bus type for the disk device. `virtio` is a para virtualized device driver that offers improved performance for virtual machines