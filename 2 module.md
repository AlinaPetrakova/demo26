</details>
 <details>
   <summary>2,3. RAID<summary>
     
- HQ-SRV
  
```tcl
lsblk
mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sd[b-d]
mdadm --detail -scan --verbose > /etc/mdadm.conf
apt-get update && apt-get install fdisk -y
echo -e "n\n\n\n\nw" | fdisk /dev/md0
mkfs.ext4 /dev/md0p1
echo "/dev/md0p1 /raid ext4 defaults 0 0" >> /etc/fstab
mkdir /raid
mount -a
apt-get install nfs-server -y
mkdir /raid/nfs
chown 99:99 /raid/nfs
chmod 777 /raid/nfs
echo "/raid/nfs 192.168.2.0/28(rw,sync,no_subtree_check)" >> /etc/exports
exportfs -a
exportfs -v
systemctl enable nfs
systemctl restart nfs
apt-get install -y chrony
echo "server 172.16.1.1 iburst prefer" >> /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources
timedatectl
```

```

- HQ-CLI

```tcl
apt-get update && apt-get install nfs-common -y
mkdir -p /mnt/nfs
echo "192.168.1.10:/raid/nfs /mnt/nfs nfs intr,soft,_netdev,x-systemd.automount 0 0" >> /etc/fstab
mount -a
mount -v
touch /mnt/nfs/test
ls /raid/nfs
apt-get install -y chrony
echo "server 172.16.1.1 iburst prefer" >> /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources
timedatectl
```
