* Ubuntu uses "/etc/netplan/*.yaml" for configuring the network.
* Run "ip a" command on machine to check which ethernet adapter os using.
  Result :
  
  ```bash
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
		link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
		inet 127.0.0.1/8 scope host lo
		valid_lft forever preferred_lft forever
		inet6 ::1/128 scope host 
		valid_lft forever preferred_lft forever
	2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
		link/ether 08:00:27:a3:a1:a4 brd ff:ff:ff:ff:ff:ff
		inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
		valid_lft 86259sec preferred_lft 86259sec
		inet6 fe80::a00:27ff:fea3:a1a4/64 scope link 
		valid_lft forever preferred_lft forever
	3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
		link/ether 08:00:27:41:d3:69 brd ff:ff:ff:ff:ff:ff
		inet 192.168.56.9/24 brd 192.168.56.255 scope global dynamic enp0s8
		valid_lft 459sec preferred_lft 459sec
		inet6 fe80::a00:27ff:fe41:d369/64 scope link 
		valid_lft forever preferred_lft forever
	4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
		link/ether 02:42:20:38:15:83 brd ff:ff:ff:ff:ff:ff
		inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
		valid_lft forever preferred_lft forever
  ```
* Os using enp0s8, so can assign it static IP.
* Check current IP of the machine and also check host machine IP using "ip a"
* Currently host machine ip is "192.168.56.1", which will be our gateway
* I chosen 192.168.56.5 as my static IP
* Copy existing yaml file from "/etc/netplan" to some where else.
* Modify "/etc/netplan/<some_name>.yaml" and add following lines
```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
	  addresses: [192.168.56.6/24]
	  dhcp4: false
```
* Modify "/etc/hosts" file and assign new static ip against hostname
* Run following command to test the changes
	- sudo netplan apply
	- ip a:w