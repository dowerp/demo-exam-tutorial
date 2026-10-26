# Настройка BR-RTR

## 1. Настройка имени и домена
```bash
conf t
hostname br-rtr
ip domain-name au-team.irpo
end
wr
```
## 2. Настройка интерфейса ISP
### Поднятие интерфейса isp
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
