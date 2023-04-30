
## Debian
* check available interfaces using
`ls /sys/class/net/`
* Open
`vi /etc/network/interfaces`
* To enable DHCP - Add following lines for the interface
	* auto enp0s8
	* iface enp0s8 inet dhcp
## CentOS
- Open the configuration file 
	- `vi /etc/sysconfig/network-scripts/ifcfg-`
- Modify and set
	- `ONBOOT=yes`
	- `BOOTPROTO=dhcp`
