# Настройка ISP

# Обязательно в конце пишем systemctl restart network

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

## 3. Настройка IP-адресов
### Проверка включения пересылки пакетов 
```bash
cat /etc/net/sysctl.conf | grep ip_forward
```
Должно вывести net.ipv4.ip_forward = 1
Если значение 0 то
```bash
vim +10 /etc/net/sysctl.conf
```
И меняем значение на 1
### Проверка IP
```bash
ip -br a
```
ens3 получает IP по DHCP от провайдера, если его нет то переходим к настройке [ens3](https://github.com/dowerp/demo-exam-tutorial/edit/main/Module-1/ISP.md#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-ens3)
### Настройка ens3
#### Файл options
```bash
vim /etc/net/ifaces/ens3/options
```
Внутри файла должно быть:
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
#### Файл options
Копируем файл options из каталога ens3
```bash
mkdir /etc/net/ifaces/ens4
cp /etc/net/ifaces/ens3/options /etc/net/ifaces/ens4/
```
Содержимое options
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
#### Файл ipv4address 
```text
172.16.1.1/28
```
### Настройка ens5
#### Файл options
Копируем options из каталога ens3
```bash
mkdir /etc/net/ifaces/ens5
cp /etc/net/ifaces/ens3/options /etc/net/ifaces/ens5/
```
Содержимое options
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
#### Файл ipv4address
```text
172.16.2.1/28
```

## 4. Настройка iptables
Если его нет то
```bash
apt-get install iptables
```
### Теперь сама настройка
```bash
iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE 
iptables-save >> /etc/sysconfig/iptables 
systemctl enable --now iptables
iptables -t nat -L
```
# Обязательно в конце пишем systemctl restart network
