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

### Поднятие интерфейсов
```bash
ip link set dev SW up
ip link set dev ens3 up
ip link set dev ens4 up
ip link set dev ens5 up
```

### Добавляем интерфейс mgmt
```bash
ovs-vsctl add-port SW mgmt999 tag=999 -- set interface mgmt999 type=internal
ip link set dev mgmt999 up
ip addr add 192.168.99.2/24 dev mgmt999
```
