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
