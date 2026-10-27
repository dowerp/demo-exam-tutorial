# Настройка HQ-CLI

## Настройка имени
```bash
hostnamectl set-hostname hq-cli.au-team.irpo
exec bash
```
## Настройка IP-адреса
### Проверяем есть ли адрес на ens3
```bash
ip -br a
```
## Настройка интерфейса
```bash
cp /etc/net/ifaces/ens18/options /etc/net/ifaces/ens3/
vim /etc/net/ifaces/ens3/options
```
Меняем содержимое файла
```text
TYPE=eth
CONFIG_WIRELESS=no
BOOTPROTO=dhcp
SYSTEMD_BOOTPROTO=dhcp4
CONFIG_IPV4=yes
DISABLES=no
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=no
```






[BR-CLI.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-CLI.md) <br>
[BR-FW.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-FW.md) <br>
[BR-RTR.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-RTR.md) <br>
[BR-SRV.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-SRV.md) <br>
[HQ-RTR.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-RTR.md) <br>
[HQ-SRV.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-SRV.md) <br>
[HQ-SW.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-SW.md) <br>
[ISP.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/ISP.md) <br>
