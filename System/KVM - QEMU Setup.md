
### 1) Install required packages

`apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst`


-   `libvirt-daemon-system` – Provides the `libvirt` daemon and a collection of tools for managing VMs.

-   `libvirt-clients` – Provides the Virsh program, a command-line interface tool for managing guests and hosts through the libvirt API.

-   `bridge-utils` – Provides programs to create and maintain Ethernet device bridges between Ethernet devices and the KVM guests on your host system.

-   `virtinst` – Provides a command-line utility for creating new KVM guests using the `virt-install` tool.

### 2) Enable libvirtd

- `sudo systemctl status libvirtd` - Use this command to check status
- `sudo systemctl enable libvirtd` - If not enabled, use this command to enable it.
### 3) Check virsh version

- `virsh --version`

### 4) Adding a User to Libvirt and KVM Group

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


### 5) Force `qemu:///system` as default.

``` sh
sudo cp -rv /etc/libvirt/libvirt.conf ~/.config/libvirt/ &&\
sudo chown  $USER ~/.config/libvirt/libvirt.conf
```

### 6) Network Configuration

Every standard `libvirt` installation provides NAT-based connectivity to virtual machines as the default virtual network. Verify that it is available with the `virsh net-list --all` command.

```sh
# virsh net-list --all
Name                 State      Autostart
-----------------------------------------
default              active     no
```

If it is missing, the example XML configuration file can be reloaded and activated:

```sh
virsh net-define /usr/share/libvirt/networks/default.xml
```

Mark the default network to automatically start:
```sh
# virsh net-autostart default
Network default marked as autostarted
# virsh net-start default
Network default started
```

Once the `libvirt` default network is running, you will see an isolated bridge device. This device does _not_ have any physical interfaces added. The new device uses NAT and IP forwarding to connect to the physical network. Do not add new interfaces.

```sh
# brctl show
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.5254007ac078	yes
```

Create `bridge.conf` to set an ACL telling QEMU that the `virbr0` interface should be whitelisted. By default all the rest is blacklisted.

```sh
$ echo "allow virbr0" > /etc/qemu/bridge.conf
$ chown root:kvm /etc/qemu/bridge.conf
$ chmod 0660 /etc/qemu/bridge.conf
```

### 7) Create VM

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

### 8) Get IP address of VM - Can be used to connect using ssh.

``` sh
$ virsh domifaddr ubuntu22042srv01
 Name       MAC address          Protocol     Address
------------------------------------------------------------------------------
 vnet1      52:54:00:9d:17:5f    ipv4         192.168.122.179/24

```

### 9) Clone image

``` sh
$ virt-clone --original ubuntu22042srv01 \
  --name ubuntu22042srv02 \
  --file /home/nilesh/KVM/images/ubuntu22042srv02.img
```

### 10) Delete image

``` sh
$ virsh list --all
$ virsh shutdown ubuntu22042srv01
$ virsh undefine ubuntu22042srv01
```

### 11) Create new VM from existing image - NOT Working / IP Conflict
``` sh
virt-install --name ubuntu22042srv02 \
--os-variant ubuntu22.04 \
--ram 3072 \
--disk /home/nilesh/KVM/images/ubuntu22042srv02.img,format=qcow2 \
--graphics vnc,listen=0.0.0.0 \
--noautoconsole \
--hvm \
--mac RANDOM \
--import
```


### 12) Check network
``` sh
$ virsh net-list
 Name      State    Autostart   Persistent
--------------------------------------------
 default   active   yes         yes

$ virsh net-info default
Name:           default
UUID:           3c96bd1b-1ecb-43e6-b941-59d2fea5b2c4
Active:         yes
Persistent:     yes
Autostart:      yes
Bridge:         virbr0
```

### Imp links
- [redhat imp commads](https://www.redhat.com/sysadmin/virsh-subcommands)
- [redhat - Networking](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/virtualization_host_configuration_and_guest_installation_guide/chap-virtualization_host_configuration_and_guest_installation_guide-network_configuration)
- [Basic Install](https://adamtheautomator.com/virsh/)1
- [KVM Networking](https://computingforgeeks.com/managing-kvm-network-interfaces-in-linux/)
- [Networking](https://apiraino.github.io/qemu-bridge-networking/)
- [CheatSheet](https://computingforgeeks.com/virsh-commands-cheatsheet/)
- [Setup](https://joshrosso.com/docs/2020/2020-05-06-linux-hypervisor-setup/)
- [virt-install examples](https://kb.novaordis.com/index.php/Virt-install_Examples)