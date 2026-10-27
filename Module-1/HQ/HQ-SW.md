# Настройка HQ-SW

## 1. Настройка имени
```bash
hostnamectl set-hostname hq-sw.au-team.irpo
exec bash
```
## 2. Работа с Open vSwitch

### Активация Open VSwitch

```bash
systemctl enable –-now openvswitch
systemctl restart network
```

### Конфигурация 
```bash
ovs-vsctl add-br SW
ovs-vsctl add-port SW ens3 trunk=100,200,999
ovs-vsctl add-port SW ens4 tag=100
ovs-vsctl add-port SW ens5 tag=200
```
