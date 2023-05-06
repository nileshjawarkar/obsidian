
### 1) Create network using "vm-manager"

![[Pasted image 20230501102719.png]]

- This is the easy way of creating the network

### 2) Edit network and add static IP's
- This step requires you to create VM's first
- Get mac address of vm using
``` sh
virsh dumpxml <VM NAME> | grep "mac "

output:

<mac address='52:54:00:9d:17:5f'/>
```

### 3) Get XML for network created in 1st step
- For me network name is kubenet. Use name you used.
``` sh
virsh net-dumpxml kubenet > kubenet.xml
```

### 4) Modify XML and add static IP's for the mac addresses
``` xml
<network>
  <name>kubenet</name>
  <uuid>17481cfe-c594-4f76-932b-309bafb2f037</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr2' stp='on' delay='0'/>
  <mac address='52:54:00:f4:2c:09'/>
  <domain name='kubenet'/>
  <ip address='192.168.200.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.200.5' end='192.168.200.15'/>
      <host mac='52:54:00:9d:17:5f' ip='192.168.200.5'/>
      <host mac='52:54:00:19:ef:b5' ip='192.168.200.6'/>
    </dhcp>
  </ip>
</network>
```

### 5) Delete network

``` sh
virsh net-destroy kubenet
virsh net-undefine kubenet
```


### 6) Create network using above XML

``` sh
virsh net-create kubenet.xml
```

### 7) Start new network

``` sh
virsh net-start kubenet
```

