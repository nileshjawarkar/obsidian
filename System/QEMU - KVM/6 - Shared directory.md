
1) Create directory on the host -
	`mkdir ~/shared`

2) Give access - 
	`sudo chmod 777 ~/shared`

3) Start VM Manager and used "Open VM details -> Add hardware" -

![[Pasted image 20230513144954.png]]

``` text
Driver : virtio-9p
Source path: /home/nilesh/shared
Target path: /shared_dir
```

4) Now on VM run following command to mount "/shared_dir" -
	`mkdir ~/shared`
	`sudo mount -t 9p -o trans=virtio /shared_dir shared/`


**That's it**

