
- HQ-SRV
  
```tcl
lsblk
mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sd[b-d]
mdadm --detail -scan --verbose > /etc/mdadm.conf
apt-get update && apt-get install fdisk -y
fdisk /dev/md0
n
w
mkfs.ext4 /dev/md0p1
vim /etc/fstab
/dev/md0p1 /raid ext4 defaults 0 0
mkdir /raid
mount -a
apt-get install nfs-server -y
mkdir /raid/nfs
chown 99:99 /raid/nfs
chmod 777 /raid/nfs
vim /etc/exports
/raid/nfs 192.168.2.0/28(rw,sync,no_subtree_check)
exportfs -a
exportfs -v
systemctl enable nfs
systemctl restart nfs
```
