# Настройка BR-SRV
## 1. Настройка имени
```bash
hostnamectl set-hostname br-srv.au-team.irpo
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



[BR-CLI.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-CLI.md) <br>
[BR-FW.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-FW.md) <br>
[BR-RTR.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-RTR.md) <br>
[HQ-CLI.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-CLI.md) <br>
[HQ-RTR.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-RTR.md) <br>
[HQ-SRV.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-SRV.md) <br>
[HQ-SW.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-SW.md) <br>
[ISP.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/ISP.md) <br>
