
## Install and start openssh server
## Ubuntu
* sudo apt-get install openssh-server
* sudo systemctl enable ssh
* sudo systemctl start ssh

## Centos
* sudo yum –y install openssh-server openssh-clients
* sudo systemctl enable sshd
* sudo systemctl start sshd
* sudo systemctl status sshd

## Configuration
* sudo vim /etc/ssh/sshd_config
* To disable root login:
	* PermitRootLogin no
* Change the SSH port to run on a non-standard port.
	* Port 2002
## Generate ssh key's on clients
* ssh-keygen OR
* ssh-keygen -t rsa -f vmuser_rsa OR
* ssh-keygen -f vmuser_rsa -m PEM

## Copy public-key to server
* ssh-copy-id -i vmuser_rsa.pub vmuser@192.168.56.3
* OR
* Manual copy
	* Login to the server using pwd 
	`ssh vmuser@192.168.56.3`
	* create following file
		* mkdir -p ~/.ssh
		* vi ~/.ssh/authorized_keys
	* copy content of .pub key to _~/.ssh/authorized_keys_
	* chmod 700 ~/.ssh 
	* chmod 600 ~/.ssh/authorized_keys 

## Login user ssh key
* ssh -i vmuser_rsa vmuser@192.168.56.3