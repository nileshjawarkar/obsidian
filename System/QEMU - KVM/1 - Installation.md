
### 1) Install required packages

`apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager`


-   `libvirt-daemon-system` – Provides the `libvirt` daemon and a collection of tools for managing VMs.

-   `libvirt-clients` – Provides the Virsh program, a command-line interface tool for managing guests and hosts through the libvirt API.

-   `bridge-utils` – Provides programs to create and maintain Ethernet device bridges between Ethernet devices and the KVM guests on your host system.

-   `virtinst` – Provides a command-line utility for creating new KVM guests using the `virt-install` tool.

### 2) Check virsh version

- `virsh --version`

