# Настройка ISP
## 1. Настройка имени
```bash
hostnamectl set-hostname isp.au-team.irpo
exec bash
```
## 2. Настройка времени
```bash
timedatectl
```
Если время Москвы не стоит то
```bash
timedatectl set-timezone Europe/Moscow
```
## Настройка IP-адресов
### Проверка IP
```bash
ip -br a
```
#### ens3(в моем случае) получает IP по DHCP от провайдера, если его нет то переходим к настройке ens3
### Настройка ens3
#### Настройка options
```bash
vim /etc/net/ifaces/ens3/options
```
#### Внутри файла должно быть:
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
### Настройка ens4
#### Копируем options из каталога ens3
```bash
mkdir /etc/net/ifaces/ens4
cp /etc/net/ifaces/ens3/options /etc/net/ifaces/ens4
```
#### Настройка options в каталоге ens4
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
