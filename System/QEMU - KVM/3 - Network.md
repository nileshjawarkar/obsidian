### 1) Configuration

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


### 2) Check network
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
