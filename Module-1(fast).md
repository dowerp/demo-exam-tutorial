[ISP]
```bash
# Имя хоста
hostname
hostnamectl set-hostname isp.au-team.irpo
hostname

# Сеть
ip a

# NAT (Masquerade)
vim /etc/net/sysctl.conf
iptables -t nat -L
iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
iptables-save >> /etc/sysconfig/iptables
systemctl enable --now iptables
iptables -t nat -L

# Время
timedatectl
```

[HQ-RTR]
```text
en
sh hostname
conf t
hostname HQ-RTR
ip domain-name au-team.irpo
end
write memory

sh ip interface brief
sh users localdb

# GRE туннель
sh interface tunnel.0
conf t
interface tunnel.0
ip address 10.10.10.1/30
ip tunnel 172.16.1.2 172.16.2.2 mode gre
end
write memory
show interface tunnel.0

# Статическая маршрутизация
sh ip route
conf t
ip route 0.0.0.0 0.0.0.0 172.16.1.1
end
write memory
show ip route

# OSPF
conf t
router ospf 0
network 10.10.100.0/27 area 0
network 10.10.200.0/28 area 0
network 10.10.30.0/29 area 0
network 10.10.10.0/30 area 0
passive-interface default
no passive-interface tunnel.0
area 0 authentication
exit
interface tunnel.0
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssword
end
write memory

# NAT
show ip nat translations
conf t
interface isp
ip nat outside
exit
interface vl100
ip nat inside
exit
interface vl200
ip nat inside
exit
interface vl999
ip nat inside
exit
ip nat pool HQ 10.10.0.1-10.10.200.254
ip nat source dynamic inside-to-outside pool HQ overload interface isp
end
write memory

# Проверка DHCP
show dhcp-server 1 detailed

# Часовой пояс
show ntp timezone
conf t
ntp timezone utc+3
end
show ntp timezone
```

[BR-RTR]
```text
en
sh hostname
conf t
hostname BR-RTR
ip domain-name au-team.irpo
end
write memory

sh ip interface brief
sh users localdb

# GRE туннель
sh interface tunnel.0
conf t
interface tunnel.0
ip address 10.10.10.2/30
ip tunnel 172.16.2.2 172.16.1.2 mode gre
end
write memory

# Статическая маршрутизация
sh ip route
conf t
ip route 0.0.0.0 0.0.0.0 172.16.2.1
end
write memory
show ip route

# OSPF
conf t
router ospf 0
network 10.20.10.0/30 area 0
network 10.10.10.0/30 area 0
passive-interface default
no passive-interface tunnel.0
no passive-interface fw
area 0 authentication
exit
interface tunnel.0
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssword
exit
interface fw
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssword
end
write memory

sh ip ospf neighbor
show ip route

# NAT
show ip nat translations
conf t
interface isp
ip nat outside
exit
interface fw
ip nat inside
exit
ip nat pool BR 10.0.0.1-10.254.254.254
ip nat source dynamic inside-to-outside pool BR overload interface isp
end
write memory

# Часовой пояс
show ntp timezone
conf t
ntp timezone utc+3
end
show ntp timezone
```

[HQ-SW]
```bash
# Имя хоста
hostname
hostnamectl set-hostname hq-sw.au-team.irpo
hostname

# Проверка сети и VLAN (OVS)
ip a
ovs-vsctl show
```

[HQ-SRV]
```bash
# Имя хоста
hostname
hostnamectl set-hostname hq-srv.au-team.irpo
hostname

# Настройка сети
ip a
vim /etc/net/ifaces/ens3/options
vim /etc/net/ifaces/ens3/ipv4address
vim /etc/net/ifaces/ens3/ipv4route
vim /etc/net/ifaces/ens3/resolv.conf
systemctl restart network
ip a

# Пользователи и Sudo
vim /etc/passwd
vim /etc/group
visudo
vim +123 /etc/sudoers

# SSH
vim /etc/openssh/sshd_config
vim +105 /etc/openssh/sshd_config
vim /etc/openssh/banner
systemctl restart sshd

# DNS (BIND)
vim /etc/bind/options.conf
vim /etc/bind/rfc1912.conf
vim /etc/bind/zone/au-team.irpo.zone
vim /etc/bind/zone/10.10.in-addr.arpa
systemctl enable --now bind.service

# Время
timedatectl
```

[HQ-CLI]
```bash
# Имя хоста
hostname
hostnamectl set-hostname hq-cli.au-team.irpo
hostname

# Сеть и время
ip a
timedatectl

# Проверка NAT
ping 172.16.1.1
```

[BR-CLI]
```bash
# Имя хоста
hostname
hostnamectl set-hostname br-cli.au-team.irpo
hostname

# Сеть
ip a
su -
mv /etc/net/ifaces/ens18 /etc/net/ifaces/ens3
vim /etc/net/ifaces/ens3/options
vim /etc/net/ifaces/ens3/ipv4address
vim /etc/net/ifaces/ens3/ipv4route
vim /etc/net/ifaces/ens3/resolv.conf
systemctl restart network
reboot

# Проверка
ip a
timedatectl
```

[BR-SRV]
```bash
# Имя хоста
hostname
hostnamectl set-hostname br-srv.au-team.irpo
hostname

# Сеть
ip a
vim /etc/net/ifaces/ens3/options
vim /etc/net/ifaces/ens3/ipv4address
vim /etc/net/ifaces/ens3/ipv4route
vim /etc/net/ifaces/ens3/resolv.conf
systemctl restart network
ip a

# Пользователи и Sudo
vim /etc/passwd
vim /etc/group
visudo
vim +123 /etc/sudoers

# SSH
vim /etc/openssh/sshd_config
vim +105 /etc/openssh/sshd_config
vim /etc/openssh/banner
systemctl restart sshd

# Время
timedatectl
```

[BR-FW]
```bash
# Веб-интерфейс доступен по адресу: https://10.20.30.1:8443
# Команды встроенного терминала:
hostname
hostnamectl set-hostname br-fw.au-team.irpo
hostname
ip a
timedatectl
```
