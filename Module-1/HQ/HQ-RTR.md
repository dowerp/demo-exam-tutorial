# Настройка HQ-RTR
## Переходим в привилегированый 
## 1. Настройка имени и домена
```bash
conf t
hostname hq-rtr
ip domain-name au-team.irpo
end
wr
```
## 2. Настройка интерфейсов ISP и VLAN
### Cоздание интерфейса ISP
```bash
interface isp
description "ISP"
ip address 172.16.1.2/28
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
### Создание VLAN
```bash
conf t
interface vl100
 ip address 10.10.100.1
 interface vl200
 ip address 10.10.200.1
 interface vl999
 ip address 10.10.30.1
end
wr
```
### Поднятие VLAN
```bash
port ge1
 service-instance ge1/v100
  encapsulation dot1q 100 exact
  rewrite pop 1
  connect ip interface vlan100
 exit

 service-instance ge1/v200
  encapsulation dot1q 200 exact
  rewrite pop 1
  connect ip interface vlan200
 exit

 service-instance ge1/v999
  encapsulation dot1q 999 exact
  rewrite pop 1
  connect ip interface vlan999
 end
wr
```
## 3. Настройка DHCP-server
### DHCP для VLAN100
```bash
conf t
ip pool VLAN100 10.10.100.2-10.10.100.254
dhcp-server 1
pool VLAN100 1
mask 24
gateway 10.10.200.1
dns 10.10.100.1
domain-name au-team.irpo
exit
exit
interface vl100
dhcp-server 1
end
wr
```
### DHCP для VLAN200
```bash
conf t
ip pool VLAN200 10.10.200.2-10.10.200.254
dhcp-server 2
pool VLAN200 1
mask 24
gateway 10.10.200.1
dns 10.10.100.2
domain-name au-team.irpo
exit
exit
interface vl200
dhcp-server 2
end
wr
```
## 4. Создание шлюза по умолчанию через IP-адрес провайдера ISP
```bash
conf t
ip route 0.0.0.0 0.0.0.0 172.16.1.1
end
wr
```
## 5. Настройка туннеля GRE
```bash
interface tunnel.0 
ip address 10.10.10.1/30 
ip tunnel 172.16.1.2 172.16.2.2 mode gre
end
wr
```
## 6. Настройка OSPF
```bash
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
wr
```
## 7. Настройка NAT
```bash
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
wr
```
## 8. Настройка timzone
```bash
conf t
ntp timezone utc+3
end
write memory
```
## Команды если требуется проверить (только в привилегированном режиме)
```bash
show ip interface brief // Проверка состояния сетевых интерфейсов
show interface tunnel.0 // Проверка состояния GRE-туннеля
show ip route // Проверка таблицы маршрутизации
show ip nat translations // Просмотр таблицы активных трансляций NAT
show dhcp-server 1 detailed // Проверка параметров встроенного сервера DHCP
```
# Всегда сохраняем настройки end > wr
