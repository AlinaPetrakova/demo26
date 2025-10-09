##  Базовая настройка устройств

- ISP
  
```tcl
hostnamectl set-hostname ISP
mkdir -p /etc/net/ifaces/{ens20,ens21,ens22}
cat > /etc/net/ifaces/ens20/options <<EOF
BOOTPROTO=dhcp
CONFIG_IPV4=yes
DISABLED=no
TYPE=eth
EOF
cp /etc/net/ifaces/ens20/options /etc/net/ifaces/ens21/options
cp /etc/net/ifaces/ens20/options /etc/net/ifaces/ens22/options
cat > /etc/net/ifaces/ens21/options <<EOF
TYPE=eth
BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
EOF
cat > /etc/net/ifaces/ens22/options <<EOF
TYPE=eth
BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
EOF
echo 172.16.1.1/28 > /etc/net/ifaces/ens21/ipv4address
echo 172.16.2.1/28 > /etc/net/ifaces/ens22/ipv4address
cat > /etc/net/sysctl.conf <<EOF
net.ipv4.ip_forward = 1
EOF
systemctl restart network
apt-get update && apt-get install iptables -y
iptables -t nat -A POSTROUTING -o ens20 -s 0/0 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
apt-get update && apt-get reinstall tzdata
timedatectl set-timezone Asia/Yekaterinburg
timedatectl
```

- HQ-RTR

```tcl
enable
conf t
hostname hq-rtr
ip domain-name au-team.irpo
write memory
interface int0
description "to isp"
ip address 172.16.1.4/28
exit
port te0
service-instance te0/int0
encapsulation untagged
exit
exit
interface int0
connect port te0 service-instance te0/int0
exit
interface int1
description "to hq-srv"
ip address 192.168.1.1/27
exit
interface int2
description "to hq-cli"
ip address 192.168.2.1/28
exit
port te1
service-instance te1/int1
encapsulation dot1q 100
rewrite pop 1
exit
service-instance te1/int2
encapsulation dot1q 200
rewrite pop 1
exit
exit
interface int1
connect port te1 service-instance te1/int1
exit
interface int2
connect port te1 service-instance te1/int2
exit
ip name-server 8.8.8.8
ip route 0.0.0.0 0.0.0.0 172.16.1.1
write
username net_admin
password P@ssw0rd
role admin
exit
write
interface int3
description "999"
ip address 192.168.99.1/29
exit
port te1
service-instance te1/int3
encapsulation dot1q 999
rewrite pop 1
exit
exit
interface int3
connect port te1 service-instance te1/int3
exit
write
interface tunnel.0
ip address 172.16.0.1/30
ip mtu 1400
ip tunnel 172.16.1.4 172.16.2.5 mode gre
ip ospf authentication-key ecorouter
exit
write
router ospf 1
network 172.16.0.1/30 area 0
network 192.168.1.0/27 area 0
network 192.168.2.0/28 area 0
passive-interface default
no passive-interface tunnel.0
area 0 authentication
exit
write
interface int1
ip nat inside
exit
interface int2
ip nat inside
exit
interface int0
ip nat outside
exit
write
ip nat pool NAT_POOL 192.168.1.1-192.168.1.254,192.168.2.1-192.168.2.254
ip nat source dynamic inside-to-outside pool NAT_POOL overload interface int0
write
ip pool cli_pool 192.168.2.10-192.168.2.10
dhcp-server 1
pool cli_pool 1
mask 255.255.255.240
gateway 192.168.2.1
dns 192.168.1.10
domain-name au-team.irpo
exit
exit
interface int2
dhcp-server 1
exit
write
ntp timezone utc+5
exit
show ntp timezone
write
```

- BR-RTR

```tcl
enable
conf t
hostname br-rtr
ip domain-name au-team.irpo
write
interface int0
description "to isp"
ip address 172.16.2.5/28
exit
port te0
service-instance te0/int0
encapsulation untagged
exit
exit
interface int0
connect port te0 service-instance te0/int0
exit
interface int1
description "to br-srv"
ip address 192.168.3.1/28
exit
port te1
service-instance te1/int1
encapsulation untagged
exit
interface int1
connect port te1 service-instance te1/int1
exit
write
ip name-server 8.8.8.8
ip route 0.0.0.0 0.0.0.0 172.16.2.1
write
username net_admin
password P@ssw0rd
role admin
exit
write
interface tunnel.0
ip address 172.16.0.2/30
ip mtu 1400
ip tunnel 172.16.2.5 172.16.1.4 mode gre
ip ospf authentication-key ecorouter
exit
write
router ospf 1
network 172.16.0.2/30 area 0
network 192.168.3.0/27 area 0
passive-interface default
no passive-interface tunnel.0
area 0 authentication
exit
write
interface int1
ip nat inside
exit
interface int0
ip nat outside
exit
write
ip nat pool NAT_POOL 192.168.3.1-192.168.3.254
ip nat source dynamic inside-to-outside pool NAT_POOL overload interface int0
write
ip name-server 8.8.8.8
write
ntp timezone utc+5
exit
write
show ntp timezone
write
```

- HQ-SRV

```tcl
hostnamectl set-hostname hq-srv.au-team.irpo
mkdir -p /etc/net/ifaces/ens20
cat > /etc/net/ifaces/ens20/options <<EOF
BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
TYPE=eth
EOF
echo 192.168.1.10/27 > /etc/net/ifaces/ens20/ipv4address
echo "default via 192.168.1.1" > /etc/net/ifaces/ens20/ipv4route
systemctl restart network
echo "nameserver 8.8.8.8" > /etc/resolv.conf
useradd remote_user -u 2026
echo "remote_user\:P@ssw0rd" | chpasswd
sed -i 's/^#\s*\(%wheel\s*ALL=(ALL:ALL)\s*NOPASSWD:\s*ALL\)/\1/' /etc/sudoers
gpasswd -a "remote_user" wheel
cat > /etc/openssh/sshd_config <<EOF
Port 2026
AllowUsers remote_user
MaxAuthTries 2
PasswordAuthentication yes
Banner /etc/openssh/banner
EOF
cat > /etc/openssh/banner <<EOF
Authorized access only
EOF
systemctl restart sshd
timedatectl set-timezone Asia/Yekaterinburg
timedatectl
apt-get update && apt-get install dnsmasq -y
systemctl enable --now dnsmasq
cat > /etc/dnsmasq.conf <<EOF
no-resolv
domain=au-team.irpo
server=8.8.8.8
interface=*
address=/hq-rtr.au-team.irpo/192.168.1.1
ptr-record=1.1.168.192.in-addr.arpa,hq-rtr.au-team.irpo
address=/docker.au-team.irpo/172.16.1.1
address=/web.au-team.irpo/172.16.2.1
address=/hq-srv.au-team.irpo/192.168.1.10
ptr-record=10.1.168.192.in-addr.arpa,hq-srv.au-team.irpo
address=/hq-cli.au-team.irpo/192.168.2.10
ptr-record=10.2.168.192.in-addr.arpa,hq-cli.au-team.irpo
address=/br-rtr.au-team.irpo/192.168.3.1
address=/br-srv.au-team.irpo/192.168.3.10
EOF
cat > /etc/hosts <<EOF
192.168.1.1 hq-rtr.au-team.irpo
EOF
systemctl restart dnsmasq
```

- BR-SRV

```tcl
hostnamectl set-hostname br-srv.au-team.irpo
mkdir -p /etc/net/ifaces/ens20
cat > /etc/net/ifaces/ens20/options <<EOF
TYPE=eth
BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
EOF
echo 192.168.3.10/28 > /etc/net/ifaces/ens20/ipv4address
echo "default via 192.168.3.1" > /etc/net/ifaces/ens20/ipv4route
echo "nameserver 8.8.8.8" > /etc/resolv.conf
systemctl restart network
useradd sshuser
echo "sshuser:P@ssw0rd" | chpasswd
echo "WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers
gpasswd -a sshuser wheel
cat > /etc/openssh/sshd_config <<EOF
Port 2024
AllowUsers sshuser
MaxAuthTries 2
PasswordAuthentication yes
Banner /etc/openssh/banner
EOF
cat > /etc/openssh/banner <<EOF
Authorized access only
EOF
systemctl restart sshd
timedatectl set-timezone Asia/Yekaterinburg
timedatectl
```

- HQ-CLI

```tcl
hostnamectl set-hostname hq-cli.au-team.irpo
mkdir -p /etc/net/ifaces/ens20
cat > /etc/net/ifaces/ens20/options <<EOF
DISABLED=no
TYPE=eth
BOOTPROTO=dhcp
CONFIG_IPV4=yes
EOF
echo "nameserver 8.8.8.8" > /etc/resolv.conf
systemctl restart network
timedatectl set-timezone Asia/Yekaterinburg
```
