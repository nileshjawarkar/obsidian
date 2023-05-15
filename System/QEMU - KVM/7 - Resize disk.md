
1) Shutdown the VM

2) Find disk location
`sudo virsh domblklist <VMNAME>`

3) Delete snapshot
`virsh snapshot-list <VMNAME>`

4) Resize disk
`sudo qemu-img resize /home/nilesh/KVM/images/rocky90srv01.qcow2 +16G`

- Avoid shrink - Can corrupt image
`sudo qemu-img resize --shrink /home/nilesh/KVM/images/rocky90srv01.qcow2 -10G`

5) VM - Extending disk partition
`sudo resize2fs /dev/sda`
