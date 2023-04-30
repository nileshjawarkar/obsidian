
- ### [Why are my cloned linux VMs fighting for the same IP](https://unix.stackexchange.com/questions/419321/why-are-my-cloned-linux-vms-fighting-for-the-same-ip)

Ans - 1
```
`systemd-networkd` uses a different method to generate the DUID than `dhclient`. `dhclient` [by default uses the link-layer address](https://manpages.debian.org/jessie/isc-dhcp-client/dhclient.8.en.html) while `systemd-networkd` uses [the contents of `/etc/machine-id`](https://www.freedesktop.org/software/systemd/man/networkd.conf.html#DUIDType=). Since the VMs were cloned, they have the same `machine-id` and the DHCP server returns the same IP for both.

To fix, replace the contents of one or both of `/etc/machine-id`. This can be anything, but deleting the file and running `systemd-machine-id-setup` will create a random `machine-id` in the same way done on machine setup.
```

Ans - 2
```
What about netplan configuration? There is an option `dhcp-configuration` that can be used as follows (excerpt from [netplan examples](https://netplan.io/examples)):

network:
  version: 2
  ethernets:
    enp3s0:
      dhcp4: yes
      dhcp-identifier: mac

by default it is using machine-id, but by changing this feature we can 'force' it not to.

Excerpt from [manpages](http://manpages.ubuntu.com/manpages/cosmic/man5/netplan.5.html)/[netplan](https://netplan.io/reference), giving more insights:

dhcp-identifier (scalar)
              When  set  to `mac'; pass that setting over to systemd-networkd to use the device's
              MAC address as a unique identifier rather than a RFC4361-compliant Client ID.  This
              has no effect when NetworkManager is used as a renderer.

```