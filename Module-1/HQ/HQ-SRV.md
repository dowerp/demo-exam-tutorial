# Настройка HQ-SRV

## 1. Настройка имени
```bash
hostnamectl set-hostname hq-srv.au-team.irpo
exec bash
```

## 2. Настройка IP-адреса 
```bash
echo "10.10.100.2/27" > /etc/net/ifaces/ens3/ipv4address
echo "default via 10.10.100.1" > /etc/net/ifaces/ens3/ipv4route
cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf // Запись конфигурации DNS-клиента
search au-team.irpo
nameserver 77.88.8.8
nameserver 127.0.0.1
EOF
systemctl restart network
```
(можно через vim по тем же путям)

## 3.
