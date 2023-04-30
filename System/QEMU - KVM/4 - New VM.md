- Note - If image OR iso path is from /home/$USER directory, then  that dir must have search permision to libvirt-qemu group 
- Following command will work, but it will give more access than required.
```sh
chmod 777 /home/nilesh
```
- `virt-install --osinfo list` - List all the OS variants
- Install Ubuntu server
``` sh
virt-install --name ubuntu22042srv01 \
--os-variant ubuntu22.04 \
--ram 3072 \
--disk /home/nilesh/KVM/images/ubuntu22042srv01.img,device=disk,bus=virtio,size=20,format=qcow2 \
--graphics vnc,listen=0.0.0.0 \
--noautoconsole \
--hvm \
--cdrom /home/nilesh/KVM/isos/ubuntu-22.04.2-live-server-amd64.iso \
--boot cdrom,hd
```
- Above command will start the installation, but we may need to connect to the VM to complete installation.
- For this we need to install  `Tiger VNC viewer`
- `virsh vncdisplay server-01` - This will list vnc address/port
```sh
$ virsh vncdisplay ubuntu22042srv01
:0
```
- Connect to server-01 using vncviewer
![[Pasted image 20230429145235.png]]
