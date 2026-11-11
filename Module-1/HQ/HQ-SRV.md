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
        forward only;
        forwarders { 77.88.8.8; }

        /*
         * Specifies which hosts are allowed to ask ordinary questions.
         */
        allow-query { any; };

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
### Вносим зоны
```bash
vim /etc/bind/local.conf
```
Содержимое:
```text
include "/etc/bind/rfc1912.conf";

// Consider adding the 1918 zones here,
// if they are not used in your organization.
//      include "/etc/bind/rfc1918.conf";

// Add other zones here
zone "au-team.irpo" {
        type master;
        file "au-team.irpo.zone";
};

zone "10.10.in-addr.arpa" {
        type master;
        file "10.10.in-addr.arpa";
};
```
### Создание файлов зон
```bash
touch /etc/bind/zone/au-team.irpo.zone
touch /etc/bind/zone/10.10.in-addr.arpa
```
### Назначение прав
```bash
chown root:named /etc/bind/zone/au-team.irpo.zone /etc/bind/zone/10.10.in-addr.arpa
chmod 640 /etc/bind/zone/au-team.irpo.zone /etc/bind/zone/10.10.in-addr.arpa
```

### Меняем настройки зон
Общая зона
```
/etc/bind/zone/au-team.irpo.zone
```

```
$TTL    1D
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                        2025110600      ; serial
                        12H             ; refresh
                        1H              ; retry
                        1W              ; expire
                        1H              ; ncache
                        )
        IN      NS      au-team.irpo.
        IN      A       10.10.100.2
br-fw   IN      A       10.10.30.10
hq-rtr  IN      A       10.10.100.1
hq-rtr  IN      A       10.10.200.1
hq-rtr  IN      A       10.10.30.1
br-rtr  IN      A       10.10.30.9
hq-srv  IN      A       10.10.100.2
hq-cli  IN      A       10.10.200.2
br-srv  IN      A       10.20.20.2
br-cli  IN      A       10.20.30.2
docker  IN      A       172.16.1.1
docker  IN      A       172.16.2.1
web     IN      A       172.16.1.1
web     IN      A       172.16.2.1
```
Обратная зона HQ-SRV
```
/etc/bind/zone/100.10.10.in-addr.arpa
```

```
$TTL    1D
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                        2025110600      ; serial
                        12H             ; refresh
                        1H              ; retry
                        1W              ; expire
                        1H              ; ncache
                        )
        IN      NS      au-team.irpo.
2       IN      PTR     hq-srv.au-team.irpo.
1       IN      PTR     hq-rtr.au-team.irpo.

```
Обратная зона HQ-CLI
```text
$TTL    1D
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                        2025110600      ; serial
                        12H             ; refresh
                        1H              ; retry
                        1W              ; expire
                        1H              ; ncache
                        )
        IN      NS      au-team.irpo.
2       IN      PTR     hq-cli.au-team.irpo.
1       IN      PTR     hq-rtr.au-team.irpo.

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
