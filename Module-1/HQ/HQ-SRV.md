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
usermpd -aG wheel sshuser
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
```
## 5. Настройка DNS
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

        listen-on { 127.0.0.1; };
        listen-on-v6 { ::1; };

        /*
         * If the forward directive is set to "only", the server will only
         * query the forwarders.
         */
        //forward only;
        //forwarders { };

        /*
         * Specifies which hosts are allowed to ask ordinary questions.
         */
        //allow-query { localnets; };

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

[BR-CLI.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-CLI.md) <br>
[BR-FW.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-FW.md) <br>
[BR-RTR.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-RTR.md) <br>
[BR-SRV.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/BR/BR-SRV.md) <br>
[HQ-CLI.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-CLI.md) <br>
[HQ-RTR.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-RTR.md) <br>
[HQ-SW.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/HQ/HQ-SW.md) <br>
[ISP.md](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-1/ISP.md) <br>
