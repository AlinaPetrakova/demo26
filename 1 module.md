##  Базовая настройка устройств

- ISP
  
```tcl
hostname ISP
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
hostname hq-r
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
ip route 0.0.0.0 0.0.0.0 172.16.1.1
write
conf t
username net_admin
password P@ssw0rd
role admin
exit
write
conf t
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
conf t
interface tunnel.0
ip address 172.16.0.1/30
ip mtu 1400
ip tunnel 172.16.4.4 172.16.5.5 mode gre
ip ospf authentication-key ecorouter
exit
write
conf t
router ospf 1
network 172.16.0.0/30 area 0
network 192.168.1.0/26 area 0
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
conf t
ip nat pool NAT_POOL 192.168.1.1-192.168.1.254 192.168.2.1-192.168.2.254
ip nat source dynamic inside-to-outside pool NAT_POOL overload interface int0
write
conf t
ping 8.8.8.8
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
conf t
ntp timezone utc+5
exit
show ntp timezone
write
```
