# Настройка HQ-SRV

## 1. Настройка имени
```bash
hostnamectl set-hostname hq-srv.au-team.irpo
exec bash
```

## 2. Настройка IP-адреса 
Проверка IP, он должен получать его по dhcp
```bash
ip -br a
```
Если ip нет, то  
```bash
echo "10.10.100.2/27" > /etc/net/ifaces/ens3/ipv4address
echo "default via 10.10.100.1" > /etc/net/ifaces/ens3/ipv4route
cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf // Запись конфигурации DNS-клиента
search au-team.irpo
nameserver 77.88.8.8
nameserver 127.0.0.1
EOF
systemctl restart network
```
(можно через vim по тем же путям)

## 3. Настройка учетной записи
```bash
useradd -m -u 2026 -s /bin/bash sshuser
useradd -m -u 1025 -s /bin/bash sshadmin
usermod -aG wheel sshuser
```
### Проверяем добавились ли они командами
```bash:
vim /etc/passwd \ проверяем в конце добавленных юзеров
vim /etc/group \ в группе wheel проверяем
### Раскомментировать sudoers
```bash
vim +123 /etc/sudoers \ убрать # у WHEEL_USERS ALL=(ALL:ALL) NOPASSWD:ALL
```

## 4. Настройка SSH
### Ставим ограничение 
```bash
vim +34 /etc/openssh/sshd_config \ убрать # и поставить цифру 2
```
### Указываем banner
```bash
vim +105 /etc/openssh/sshd_config \ убрать # и вместо none ставим /etc/openssh/banner
```
### Конфигурация banner
```bash
vim /etc/openssh/banner \ вносим Authorized access only
systemctl restart network
```
## 5. Настройка DNS
### Установка bind(если его нет)
```bash
apt-get install bind
```
### Заходим в настройки bind
```bash
vim /etc/bind/options.conf
```

### Меняем listen-on, forwaders и allow-query
```text
options {
        version "unknown";
        directory "/etc/bind/zone";
        dump-file "/var/run/named/named_dump.db";
        statistics-file "/var/run/named/named.stats";
        recursing-file "/var/run/named/named.recursing";
        secroots-file "/var/run/named/named.secroots";

        // disables the use of a PID file
        pid-file none;

        /*
         * Oftenly used directives are listed below.
         */

        listen-on { 10.10.100.2; 127.0.0.1; };
        listen-on-v6 { none; };

        /*
         * If the forward directive is set to "only", the server will only
         * query the forwarders.
         */
        //forward only;
        //forwarders { 77.88.8.8; }

        /*
         * Specifies which hosts are allowed to ask ordinary questions.
         */
        //allow-query { any; };

        /*
         * This lets "allow-query" be used to specify the default zone access
         * level rather than having to have every zone override the global
         * value. "allow-query-cache" can be set at both the options and view
         * levels. If "allow-query-cache" is not set then "allow-recursion" is
         * used if set, otherwise "allow-query" is used if set unless
         * "recursion no;" is set in which case "none;" is used, otherwise the
         * default (localhost; localnets;) is used.
         */
        //allow-query-cache { localnets; };
```
### Меняем настройки зоны
```
/etc/bind/zone/au-team.irpo.zone
```

```
$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. root.au-team.irpo. ( // SOA-запись зоны прямого просмотра
            2024010101 3600 1800 604800 86400 ) // Серийный номер и таймеры зоны
    IN  NS  hq-srv.au-team.irpo. // NS-запись авторитетного сервера

hq-rtr  IN  A   172.16.1.2 // A-запись маршрутизатора HQ-RTR
br-rtr  IN  A   172.16.2.2 // A-запись маршрутизатора BR-RTR
hq-srv  IN  A   10.10.100.2 // A-запись сервера HQ-SRV
hq-cli  IN  A   10.10.200.2 // A-запись рабочей станции HQ-CLI
br-srv  IN  A   10.20.20.2 // A-запись сервера BR-SRV
br-cli  IN  A   10.20.30.2 // A-запись рабочей станции BR-CLI
br-fw   IN  A   10.20.10.2 // A-запись межсетевого экрана BR-FW
web     IN  A   172.16.1.1 // A-запись внешнего интерфейса ISP (к HQ)
docker  IN  A   172.16.2.1 // A-запись внешнего интерфейса ISP (к BR)
```
```
/etc/bind/zone/10.10.in-addr.arpa
```
```
$TTL 1D
@   IN  SOA au-team.irpo. root.au-team.irpo. ( // SOA-запись зоны обратного просмотра
            2025110600 12H 1H 1W 1H ) // Серийный номер и таймеры
    IN  NS  au-team.irpo. // NS-запись зоны

1.100   IN  PTR hq-rtr.au-team.irpo. // PTR-запись адреса 10.10.100.1
2.100   IN  PTR hq-srv.au-team.irpo. // PTR-запись адреса 10.10.100.2
2.200   IN  PTR hq-cli.au-team.irpo. // PTR-запись адреса 10.10.200.2
```
### Запускаем DNS

```
systemctl enable --now bind.service
systemctl restart bind.service
```

[BR-CLI.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-CLI.md) <br>
[BR-FW.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-FW.md) <br>
[BR-RTR.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-RTR.md) <br>
[BR-SRV.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-SRV.md) <br>
[HQ-CLI.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-CLI.md) <br>
[HQ-RTR.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-RTR.md) <br>
[HQ-SW.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-SW.md) <br>
[ISP.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/ISP.md) <br>
