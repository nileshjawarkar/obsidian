1) Down the latest installation ISO and boot the from the ISO.
	- Write ISO to USB
	- For VM, attach ISO to optical drive
2) installation 

	- set keybord
		* Default keybord is US-EN. This will work for most of the users.
		* For setting other keybords
			+ ls /usr/share/kbd/keymaps/**/*.map.gz
			+ loadkeys de-latin1
	- Update the system clock
		* timedatectl set-ntp true
		* check status
			+ timedatectl status
	- Verify boot mode
		* ls /sys/firmware/efi/efivars
		* If the command shows the directory without error, then the system is booted in UEFI mode
	- Check internet
		* ping 8.8.8.8
	- Disk partition
		* fdisk -l
			- This will display all the disk available
		* fdisk /dev/sda
			- Select the disk for the partition
		* Create disk lable
			- o : for DOS partition
			- g : UEFI partition
		* Create partition
			- n : Add new partition
				+ create partition for swap and root
			- t : change type of the partition
				+ by default partitions are created as Linux filesystem type
				+ For swap we need to change it.
			- w : write changes to disk
		* Format the partition
			- mkswap /dev/sda1
			- mkfs.ext4 /dev/sda2
	- mount /dev/sda2 /mnt
	- swapon /dev/sda1
	- packstrap /mnt base linux linux-firmware grub git networkmanager sudo vim
	- genfstab -U /mnt >> /mnt/etc/fstab
	- arch-chroot /mnt
	- ln -sf /usr/share/zoneinfo/Region/City /etc/localtime 
		* Set timezone
	- hwclock --systohc
	- vim /etc/locale.gen 
		* Un-comment required locale
	- locale-gen
	- vim /etc/locale.conf 
		* Add following line to the file
		* LANG=en_US.UTF-8
	- [optional] Configure keybord 
		* vim /etc/vconsole.conf
		* KEYMAP=de-latin1
	- Network configuration
		* set hostname
			- vim /etc/hostname
			- Add hostname to the file
		* configure localhost
			- vim /etc/hosts
			- Add following test to the file
			```bash
			127.0.0.1	localhost
			::1			localhost
			127.0.1.1	myhostname.localdomain	myhostname
			```
	- Creating a new initramfs
		* mkinitcpio -P
	- Set passward for user
		* passwd
	- create new user
		* useradd vmuser
		* mkdir /home/vmuser
		* passwd
		* usermod -aG wheel,audio,video,storage,optical vmuser
	- Setup bootloader
		* grub-install --target=i386-pc /dev/sdX
		* grub-mkconfig -o /boot/grub/grub.cfg
	- Enable network manager
		* systemctl enable NetworkManager
	- exit
	- umount -f /mnt
	- shutdown -f now
	  