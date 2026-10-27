# Настройка BR-RTR

## 1. Настройка имени и домена
```bash
conf t
hostname br-rtr
ip domain-name au-team.irpo
end
wr
```
## 2. Настройка интерфейсов 
### Создание интерфейса isp
```bash
interface isp
description "ISP"
ip address 172.16.2.2/28
end
wr
```
### Поднятие интерфейса isp
```bash
conf t
port ge0
service-instance ge0/isp
encapsulation untagged
connect ip interface isp
end
wr
```
### Создание интерфейса fw
```
interface fw
description "br-fw"
ip address 10.20.10.1/30
end
wr
```
### Поднятие интерфейса fw
```
conf t
port ge1
service-instance ge1/fw
encapsulation untagged
connect ip interface fw
end
wr
```
## 3. Настройка статического маршрута по умолчанию через ISP
```
ip route 0.0.0.0 0.0.0.0 172.16.2.1
```
## 4.  Настройка туннеля GRE
```
interface tunnel.0
ip address 10.10.10.2/30
ip tunnel 172.16.2.2 172.16.1.2 mode gre
exit
```
## 5. Динамическая маршрутизация OSPF
```
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
exit
```
## 6. Динамическая трансляция адресов (NAT)
```
interface isp
ip nat outside
exit
interface fw
ip nat inside
exit
ip nat pool BR 10.0.0.1-10.254.254.254
ip nat source dynamic inside-to-outside pool BR overload interface isp
```
## 7. Часовой пояс и сохранение
```
ntp timezone utc+3
end
wr
```
## 8. Проверочные команды на BR-RTR
```
show ip ospf neighbor 
show ip route 
show ip nat translations
``` 
