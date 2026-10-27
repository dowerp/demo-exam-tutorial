# Настройка BR-CLI

## Настройка имени
```bash
hostnamectl set-hostname br-cli.au-team.irpo
exec bash
```

## Настройка ip
```bash
cp /etc/net/ifaces/ens18/options /etc/net/ifaces/ens3
vim /etc/net/ifaces/ens3/options
```
Внутри файла должно быть
```text
TYPE=eth
CONFIG_WIRELESS=no
BOOTPROTO=static
SYSTEMD_BOOTPROTO=static
CONFIG_IPV4=yes
DISABLES=no
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=no
```

### Задаем IP
```bash
echo "10.20.30.2/26" > /etc/net/ifaces/ens3/ipv4addpess
echo "default via 10.20.30.1" > /etc/net/ifaces/ens3/ipv4route
echo << "EOF" > /etc/net/ifaces/ens3/resolv.conf
search au-team.irpo
nameserver 10.10.100.2
systemctl restart network
```



[BR-FW.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-FW.md) <br>
[BR-RTR.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-RTR.md) <br>
[BR-SRV.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-SRV.md) <br>
[HQ-CLI.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-CLI.md) <br>
[HQ-RTR.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-RTR.md) <br>
[HQ-SRV.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-SRV.md) <br>
[HQ-SW.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-SW.md) <br>
[ISP.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/ISP.md) <br>
