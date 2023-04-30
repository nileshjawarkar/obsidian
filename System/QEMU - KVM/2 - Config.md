### 1) Enable libvirtd

- `sudo systemctl status libvirtd` - Use this command to check status
- `sudo systemctl enable libvirtd` - If not enabled, use this command to enable it.


### 2) Adding a User to Libvirt and KVM Group

``` sh
$ cat /etc/group | grep kvm
kvm:x:109:
$ cat /etc/group | grep libvirt
libvirt:x:140:nilesh
libvirt-qemu:x:64055:libvirt-qemu,nilesh
libvirt-dnsmasq:x:141:
```

-  Add your users to kvm and libvirt group
``` sh
sudo adduser nilesh kvm
sudo adduser nilesh libvirt
```

- We can check groups for the users using
``` sh
$ id $USER
uid=1000(nilesh) gid=137(vboxusers) groups=137(vboxusers),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),109(kvm),115(lpadmin),135(sambashare),140(libvirt),64055(libvirt-qemu)
```


### 3) Force `qemu:///system` as default.

``` sh
sudo cp -rv /etc/libvirt/libvirt.conf ~/.config/libvirt/ &&\
sudo chown  $USER ~/.config/libvirt/libvirt.conf
```
