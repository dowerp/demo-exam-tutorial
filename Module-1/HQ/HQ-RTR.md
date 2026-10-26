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
## 2. Настройка VLAN
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
## 3. Создание шлюза по умолчанию через IP-адрес провайдера ISP
```bash
conf t
ip route 0.0.0.0 0.0.0.0 172.16.1.1
end
wr
```
## 4. Настройка туннеля GRE
```bash
interface tunnel.0 
ip address 10.10.10.1/30 
ip tunnel 172.16.1.2 172.16.2.2 mode gre
end
wr
```
## 5. Настройка OSPF
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
## 6. Настройка NAT
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
## 7. Настройка timzone
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
