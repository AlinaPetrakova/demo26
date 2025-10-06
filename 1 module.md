## ISP
```tcl 
hostnamectl set-hostname ISP 
exec bash
mkdir -p /etc/net/ifaces/{ens20,ens21,ens22}
cat > /etc/net/ifaces/ens20/options <<'EOF'
TYPE=eth
BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
EOF

```
