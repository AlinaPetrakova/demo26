</details>
 <details>
   <summary>2,3 Конфигурация файлового хранилища на сервере HQ-SRV</summary>

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
```

</details>
 <details>
   <summary>4. Настройка службы сетевого времени на базе сервиса chrony</summary>

- ISP

 ```tcl
apt-get install -y chrony
cat > /etc/chrony.conf <<EOF
server 127.0.0.1 iburst prefer
hwtimestamp *
local stratum 5
allow 0/0
EOF
systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources
chronyc tracking | grep Stratum
```

- HQ-SRV

```tcl
apt-get install -y chrony
echo "server 172.16.1.1 iburst prefer" >> /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
```

- HQ-CLI

```tcl
apt-get install -y chrony
echo "server 172.16.1.1 iburst prefer" >> /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
```

- HQ-RTR

```tcl
enable
configure terminal
ntp server 172.16.1.1
ntp timezone utc+5
write memory
ip nat source static tcp 172.16.1.4 8080 192.168.1.10 80
ip nat source static tcp 172.16.1.4 2026 192.168.1.10 2026
write
```

- BR-RTR

```tcl
enable
configure terminal
ntp server 172.16.2.1
ntp timezone utc+5
write memory
ip nat source static tcp 172.16.2.5 8080 192.168.3.10 8080
ip nat source static tcp 172.16.2.5 2026 192.168.3.10 2026
write
```

- BR-SRV

```tcl
apt-get update && apt-get install chrony -y
echo "server 172.16.1.1 iburst prefer" >> /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
```

</details>
 <details>
   <summary>4. Настройка службы сетевого времени на базе сервиса chrony</summary>

- HQ-RTR

```tcl
ip nat source static tcp 172.16.1.4 8080 192.168.1.10 80
ip nat source static tcp 172.16.1.4 2026 192.168.1.10 2026
write
```

- BR-RTR

```tcl
ip nat source static tcp 172.16.2.5 8080 192.168.3.10 8080
ip nat source static tcp 172.16.2.5 2026 192.168.3.10 2026
write
```
