# Справочник команд для решения Модуля 1: Настройка сетевой инфраструктуры

## 1. Сводная таблица IP-адресации и параметров Модуля 1

| Устройство | Интерфейс | IP-адрес / Префикс | Шлюз по умолчанию | Примечание |
|---|---|---|---|---|
| **ISP** | ens3 (WAN) | DHCP | — | Выход в Internet |
| | ens4 (к HQ) | `172.16.1.1/28` | — | Локальная сеть ISP-HQ |
| | ens5 (к BR) | `172.16.2.1/28` | — | Локальная сеть ISP-BR |
| **HQ-RTR** | isp | `172.16.1.2/28` | `172.16.1.1` | WAN-интерфейс |
| | v1100 | `10.10.100.1/27` | — | VLAN 100 (HQ-SRV) |
| | v1200 | `10.10.200.1/28` | — | VLAN 200 (HQ-CLI, DHCP) |
| | v1999 | `10.10.30.1/29` | — | VLAN 999 (Управление) |
| | tunnel.0 | `10.10.10.1/30` | — | GRE-туннель до BR |
| **BR-RTR** | isp | `172.16.2.2/28` | `172.16.2.1` | WAN-интерфейс |
| | fw | `10.20.10.1/30` | — | Линк к BR-FW |
| | tunnel.0 | `10.10.10.2/30` | — | GRE-туннель до HQ |
| **BR-FW** | Leth2 (WAN) | `10.20.10.2/30` | `10.20.10.1` | Линк к BR-RTR |
| | Leth3 (SRV) | `10.20.20.1/28` | — | Локальная сеть BR-SRV |
| | Leth4 (CLI) | `10.20.30.1/29` | — | Локальная сеть BR-CLI |
| **HQ-SRV** | ens3 | `10.10.100.2/27` | `10.10.100.1` | Сервер HQ, DNS-сервер |
| **HQ-CLI** | ens3 | DHCP (`10.10.200.2/28`) | `10.10.200.1` | Клиентская станция HQ |
| **BR-SRV** | ens3 | `10.20.20.2/28` | `10.20.20.1` | Сервер филиала BR |
| **BR-CLI** | ens3 | `10.20.30.2/29` | `10.20.30.1` | Клиентская станция BR |

---

## 2. Маршрутизатор провайдера (ISP)

```bash
hostnamectl set-hostname isp.au-team.irpo // Установка имени узла и доменного суффикса
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/g' /etc/net/sysctl.conf // Включение маршрутизации транзитных IPv4 пакетов
systemctl restart network // Применение параметров сетевой подсистемы
apt-get update && apt-get install -y iptables // Обновление репозиториев и установка утилиты iptables
iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE // Динамическая трансляция портов (PAT) наружу через WAN-интерфейс
iptables-save >> /etc/sysconfig/iptables // Сохранение правил сетевого экрана в автозагрузку
systemctl enable --now iptables // Активация и немедленный запуск службы iptables
timedatectl set-timezone Europe/Moscow // Установка часового пояса Московского региона
```

---

## 3. Центральный маршрутизатор (HQ-RTR — EcoRouterOS)

```bash
enable // Вход в привилегированный режим
configure terminal // Переход в режим глобальной конфигурации
hostname HQ-RTR // Установка сетевого имени маршрутизатора
ip domain-name au-team.irpo // Задание локального доменного имени
username net_admin // Создание учетной записи администратора сети
password P@ssword // Установка пароля для учетной записи
role admin // Назначение максимальных привилегий администратора
exit // Выход из режима настройки пользователя
ip route 0.0.0.0/0 172.16.1.1 // Создание статического маршрута по умолчанию в сторону ISP
interface tunnel.0 // Создание логического туннельного интерфейса
ip address 10.10.10.1/30 // Назначение IP-адреса туннеля со стороны HQ
ip tunnel 172.16.1.2 172.16.2.2 mode gre // Настройка инкапсуляции GRE между внешними адресами HQ и BR
exit // Выход из настройки туннеля
router ospf 0 // Запуск процесса динамической маршрутизации OSPF
network 10.10.100.0/27 area 0 // Анонс подсети VLAN 100 в магистральную область 0
network 10.10.200.0/28 area 0 // Анонс подсети VLAN 200 в магистральную область 0
network 10.10.30.0/29 area 0 // Анонс подсети управления VLAN 999 в магистральную область 0
network 10.10.10.0/30 area 0 // Анонс подсети GRE-туннеля в область 0
passive-interface default // Блокировка рассылки сообщений OSPF Hello на всех портах
no passive-interface tunnel.0 // Разрешение обмена OSPF только через защищаемый GRE-туннель
area 0 authentication // Активация проверки подлинности пакетов OSPF в области 0
exit // Выход из конфигурации процесса OSPF
interface tunnel.0 // Вход в интерфейс GRE-туннеля
ip ospf authentication message-digest // Включение криптографической аутентификации MD5
ip ospf message-digest-key 1 md5 P@ssword // Настройка ключа MD5 под номером 1 с паролем P@ssword
exit // Выход из настройки туннельного интерфейса
interface isp // Вход в конфигурацию интерфейса в сторону провайдера
ip nat outside // Назначение интерфейса внешней зоной NAT
exit // Выход из интерфейса isp
interface vl100 // Вход в конфигурацию интерфейса серверов
ip nat inside // Назначение интерфейса внутренней зоной NAT
exit // Выход из интерфейса vl100
interface vl200 // Вход в конфигурацию интерфейса клиентов
ip nat inside // Назначение интерфейса внутренней зоной NAT
exit // Выход из интерфейса vl200
interface vl999 // Вход в конфигурацию интерфейса управления
ip nat inside // Назначение интерфейса внутренней зоной NAT
exit // Выход из интерфейса vl999
ip nat pool HQ 10.10.0.1-10.10.200.254 // Формирование пула локальных IP-адресов филиала HQ
ip nat source dynamic inside-to-outside pool HQ overload interface isp // Включение динамической трансляции (PAT) пула HQ в адрес порта isp
ip pool VLAN200 10.10.200.2-10.10.200.14 // Создание пула адресов DHCP с исключением адреса шлюза
dhcp-server 1 // Создание инстанса сервера DHCP с идентификатором 1
pool VLAN200 1 // Привязка пула адресов VLAN200 к экземпляру сервера 1
mask 28 // Передача клиентам маски подсети /28 (255.255.255.240)
gateway 10.10.200.1 // Передача клиентам адреса шлюза по умолчанию
dns 10.10.100.2 // Передача клиентам IP-адреса локального DNS-сервера HQ-SRV
domain-name au-team.irpo // Передача клиентам DNS-суффикса домена
exit // Выход из конфигурации пула DHCP
exit // Выход из конфигурации DHCP-сервера
interface vl200 // Переход в клиентский сабинтерфейс
dhcp-server 1 // Активация службы DHCP-сервера на клиентском интерфейсе
exit // Выход из интерфейса vl200
ntp timezone utc+3 // Установка системного часового пояса (MSK, UTC+3)
end // Возврат в привилегированный режим EXEC
write memory // Запись текущей конфигурации в энергонезависимую память
```

---

## 4. Маршрутизатор филиала (BR-RTR — EcoRouterOS)

```bash
enable // Вход в привилегированный режим
configure terminal // Переход в режим глобальной конфигурации
hostname BR-RTR // Установка сетевого имени маршрутизатора филиала
ip domain-name au-team.irpo // Задание доменного суффикса
username net_admin // Создание учетной записи сетевого администратора
password P@ssword // Установка пароля администратора
role admin // Выдача максимальных прав администратора
exit // Выход из настройки пользователя
ip route 0.0.0.0/0 172.16.2.1 // Настройка маршрута по умолчанию в сторону ISP
interface tunnel.0 // Создание туннельного интерфейса GRE
ip address 10.10.10.2/30 // Назначение IP-адреса туннеля со стороны филиала BR
ip tunnel 172.16.2.2 172.16.1.2 mode gre // Настройка адресов источника и назначения GRE-туннеля
exit // Выход из настройки туннеля
router ospf 0 // Запуск процесса маршрутизации OSPF
network 10.20.10.0/30 area 0 // Анонс подсети соединения с межсетевым экраном BR-FW
network 10.10.10.0/30 area 0 // Анонс подсети туннеля GRE в область 0
passive-interface default // Блокировка рассылки OSPF Hello на всех интерфейсах
no passive-interface tunnel.0 // Разрешение установления соседства через GRE-туннель
no passive-interface fw // Разрешение обмена OSPF с межсетевым экраном BR-FW
area 0 authentication // Включение проверки подлинности OSPF в области 0
exit // Выход из конфигурации OSPF
interface tunnel.0 // Вход в конфигурацию туннельного интерфейса
ip ospf authentication message-digest // Включение аутентификации MD5 на туннеле
ip ospf message-digest-key 1 md5 P@ssword // Настройка ключа OSPF MD5 с паролем P@ssword
exit // Выход из интерфейса tunnel.0
interface fw // Вход в интерфейс соединения с межсетевым экраном
ip ospf authentication message-digest // Включение аутентификации MD5 на стыке с файрволом
ip ospf message-digest-key 1 md5 P@ssword // Задание ключа OSPF MD5 для связи с BR-FW
exit // Выход из интерфейса fw
interface isp // Вход в интерфейс подключения к провайдеру
ip nat outside // Назначение интерфейса внешней зоной NAT
exit // Выход из интерфейса isp
interface fw // Вход в интерфейс подключения к филиалу
ip nat inside // Назначение интерфейса внутренней зоной NAT
exit // Выход из интерфейса fw
ip nat pool BR 10.0.0.1-10.254.254.254 // Создание пула адресов внутренних подсетей филиала
ip nat source dynamic inside-to-outside pool BR overload interface isp // Включение PAT для внутренних сетей филиала через адрес порта isp
ntp timezone utc+3 // Установка часового пояса UTC+3 (Москва)
end // Переход в привилегированный режим
write memory // Сохранение конфигурации в память маршрутизатора
```

---

## 5. Коммутатор сегмента HQ (HQ-SW — Open vSwitch)

```bash
hostnamectl set-hostname hq-sw.au-team.irpo // Установка имени узла коммутатора
apt-get update && apt-get install -y openvswitch // Обновление пакетов и установка ПО Open vSwitch
systemctl enable --now openvswitch // Включение автозапуска и старт службы Open vSwitch
ovs-vsctl add-br hq-sw // Создание виртуального моста коммутации hq-sw
ovs-vsctl add-port hq-sw ens3 trunk=100,200,999 // Настройка магистрального транкового порта к HQ-RTR для VLAN 100, 200, 999
ovs-vsctl add-port hq-sw ens4 tag=100 // Настройка порта доступа для сервера HQ-SRV в VLAN 100
ovs-vsctl add-port hq-sw ens5 tag=200 // Настройка порта доступа для рабочей станции HQ-CLI в VLAN 200
ovs-vsctl show // Проверка созданной структуры портов и тегов моста
```

---

## 6. Сервер главного офиса (HQ-SRV — ALT Server)

### 6.1. Сеть и учетные записи
```bash
hostnamectl set-hostname hq-srv.au-team.irpo // Назначение FQDN имени сервера
mkdir -p /etc/net/ifaces/ens3 // Создание директории настроек сетевого адаптера ens3
cat << 'EOF' > /etc/net/ifaces/ens3/options // Запись базовых параметров адаптера в options
TYPE=eth
BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
EOF
echo "10.10.100.2/27" > /etc/net/ifaces/ens3/ipv4address // Назначение статического IPv4-адреса и маски
echo "default via 10.10.100.1" > /etc/net/ifaces/ens3/ipv4route // Установка шлюза по умолчанию на HQ-RTR
cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf // Запись настроек DNS-резолвера
search au-team.irpo
nameserver 77.88.8.8
nameserver 127.0.0.1
EOF
systemctl restart network // Применение сетевых настроек и перезапуск сети
useradd sshuser -u 2026 // Создание пользователя sshuser с фиксированным UID 2026
echo "sshuser:P@ssword" | chpasswd // Установка заданного пароля для пользователя sshuser
usermod -aG wheel sshuser // Добавление пользователя sshuser в группу wheel
echo "sshuser ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers // Разрешение выполнения sudo без ввода пароля для sshuser
timedatectl set-timezone Europe/Moscow // Установка системного часового пояса
```

### 6.2. Настройка SSH
```bash
sed -i 's/^#*Port .*/Port 2026/' /etc/openssh/sshd_config // Смена стандартного порта SSH на порт 2026
echo "AllowUsers sshuser" >> /etc/openssh/sshd_config // Ограничение доступа по SSH только пользователю sshuser
sed -i 's/^#*MaxAuthTries .*/MaxAuthTries 2/' /etc/openssh/sshd_config // Ограничение количества попыток ввода пароля до двух
sed -i 's|^#*Banner .*|Banner /etc/openssh/banner|' /etc/openssh/sshd_config // Включение пути к файлу приветственного баннера
echo "Authorized access only" > /etc/openssh/banner // Создание текста обязательного баннера безопасности
systemctl restart sshd // Перезапуск демона SSH для применения политик
```

### 6.3. Настройка службы DNS (BIND)
```bash
apt-get update && apt-get install -y bind bind-utils // Установка DNS-сервера BIND и утилит диагностики
cat << 'EOF' > /etc/bind/options.conf // Конфигурация параметров прослушивания и пересылки DNS
options {
    version "unknown";
    directory "/etc/bind/zone";
    listen-on { 127.0.0.1; 10.10.100.2; };
    listen-on-v6 { none; };
    forwarders { 77.88.8.8; };
    allow-query { any; };
    allow-recursion { any; };
};
EOF
cat << 'EOF' >> /etc/bind/rfc1912.conf // Объявление зон прямого и обратного просмотра
zone "au-team.irpo" {
    type master;
    file "au-team.irpo.zone";
    allow-update { none; };
};

zone "10.10.in-addr.arpa" {
    type master;
    file "10.10.in-addr.arpa";
    allow-update { none; };
};
EOF
cat << 'EOF' > /etc/bind/zone/au-team.irpo.zone // Наполнение ресурсными записями зоны прямого просмотра
$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. root.au-team.irpo. (
            2026040501 3600 1800 604800 86400 )
    IN  NS  hq-srv.au-team.irpo.

hq-rtr  IN  A   10.10.100.1
hq-srv  IN  A   10.10.100.2
hq-cli  IN  A   10.10.200.2
br-rtr  IN  A   172.16.2.2
br-fw   IN  A   10.20.10.2
br-srv  IN  A   10.20.20.2
br-cli  IN  A   10.20.30.2
docker  IN  A   172.16.1.1
web     IN  A   172.16.2.1
EOF
cat << 'EOF' > /etc/bind/zone/10.10.in-addr.arpa // Наполнение ресурсными записями зоны обратного просмотра
$TTL 86400
@   IN  SOA au-team.irpo. root.au-team.irpo. (
            2026040501 12H 1H 1W 1H )
    IN  NS  au-team.irpo.

1.100   IN  PTR hq-rtr.au-team.irpo.
2.100   IN  PTR hq-srv.au-team.irpo.
2.200   IN  PTR hq-cli.au-team.irpo.
EOF
systemctl enable --now bind.service // Добавление в автозапуск и старт DNS-сервера BIND
```

---

## 7. Сервер филиала (BR-SRV — ALT Server)

```bash
hostnamectl set-hostname br-srv.au-team.irpo // Установка сетевого FQDN имени сервера филиала
mkdir -p /etc/net/ifaces/ens3 // Создание директории сетевого интерфейса ens3
cat << 'EOF' > /etc/net/ifaces/ens3/options // Запись параметров сетевого интерфейса
TYPE=eth
BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
EOF
echo "10.20.20.2/28" > /etc/net/ifaces/ens3/ipv4address // Назначение статического адреса в подсети серверов филиала
echo "default via 10.20.20.1" > /etc/net/ifaces/ens3/ipv4route // Установка маршрута по умолчанию на шлюз BR-FW
cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf // Указание доменного суффикса и DNS-сервера HQ-SRV
search au-team.irpo
nameserver 10.10.100.2
EOF
systemctl restart network // Применение сетевых настроек
useradd sshuser -u 2026 // Создание пользователя sshuser с идентификатором 2026
echo "sshuser:P@ssword" | chpasswd // Задание пароля пользователя sshuser
usermod -aG wheel sshuser // Добавление пользователя в привилегированную группу wheel
echo "sshuser ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers // Включение беспарольного вызова команд через sudo
sed -i 's/^#*Port .*/Port 2026/' /etc/openssh/sshd_config // Перевод службы SSH на порт 2026
echo "AllowUsers sshuser" >> /etc/openssh/sshd_config // Разрешение авторизации по SSH только для sshuser
sed -i 's/^#*MaxAuthTries .*/MaxAuthTries 2/' /etc/openssh/sshd_config // Ограничение попыток входа до двух
sed -i 's|^#*Banner .*|Banner /etc/openssh/banner|' /etc/openssh/sshd_config // Активация пути к баннеру в конфигурации sshd
echo "Authorized access only" > /etc/openssh/banner // Формирование текста приветственного баннера безопасности
systemctl restart sshd // Перезапуск службы SSH для применения параметров защиты
timedatectl set-timezone Europe/Moscow // Настройка часового пояса
```

---

## 8. Клиентская машина главного офиса (HQ-CLI — ALT Workstation)

```bash
hostnamectl set-hostname hq-cli.au-team.irpo // Установка имени клиентской рабочей станции HQ
timedatectl set-timezone Europe/Moscow // Настройка часового пояса
ip a show ens3 // Проверка успешного получения IP-адреса по протоколу DHCP (10.10.200.X/28)
ip r // Проверка автоматической установки маршрута по умолчанию через HQ-RTR (10.10.200.1)
cat /etc/resolv.conf // Проверка получения DNS-сервера (10.10.100.2) и поискового домена (au-team.irpo)
```

---

## 9. Клиентская машина филиала (BR-CLI — ALT Workstation)

```bash
hostnamectl set-hostname br-cli.au-team.irpo // Установка имени клиентской рабочей станции BR
mv /etc/net/ifaces/ens18 /etc/net/ifaces/ens3 2>/dev/null || true // Переименование каталога интерфейса в соответствии с реальным именем адаптера
cat << 'EOF' > /etc/net/ifaces/ens3/options // Запись конфигурационных параметров интерфейса
TYPE=eth
BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
EOF
echo "10.20.30.2/29" > /etc/net/ifaces/ens3/ipv4address // Назначение статического IP-адреса рабочей станции филиала
echo "default via 10.20.30.1" > /etc/net/ifaces/ens3/ipv4route // Установка шлюза по умолчанию через BR-FW
cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf // Указание доменного суффикса и адреса DNS-сервера HQ-SRV
search au-team.irpo
nameserver 10.10.100.2
EOF
systemctl restart network // Перезапуск сети для применения адресации и маршрутов
timedatectl set-timezone Europe/Moscow // Установка часового пояса
```

---

## 10. Межсетевой экран филиала (BR-FW — Ideco NGFW)

### 10.1. Консольные команды (Терминал / SSH)
```bash
hostnamectl set-hostname br-fw.au-team.irpo // Установка полного доменного имени межсетевого экрана
timedatectl set-timezone Europe/Moscow // Установка системного часового пояса
ip -4 a // Проверка интерфейсов: Leth2 (10.20.10.2/30), Leth3 (10.20.20.1/28), Leth4 (10.20.30.1/29)
```

### 10.2. Действия в Веб-интерфейсе Ideco NGFW (`https://10.20.30.1:8443`)
* **Авторизация:** Логин `Admin`, Пароль `IdecoP@ssword`.
* **Сервисы -> Маршрутизация -> OSPF:**
  * Перевести переключатель OSPF в положение **Работает**.
  * **Router ID:** `10.20.30.1` // Задание уникального идентификатора маршрутизатора
  * **Аутентификация соседей:** Выбрать `MD5` // Включение алгоритма проверки подлинности
  * **Key ID:** `1` // Идентификатор ключа (соответствует настройке на BR-RTR)
  * **Пароль:** `P@ssword` // Секретный ключ аутентификации
  * Нажать **Сохранить**.
* **Вкладка «Локальные интерфейсы» -> Кнопка «+ Добавить»:**
  * **Интерфейс:** Выбрать интерфейс стыка с BR-RTR (`Leth2` / `oif0`)
  * **Область (Area):** `0` // Подключение интерфейса к магистральной области OSPF
  * **Тип области:** `Normal`
  * **Стоимость:** `1`
  * Нажать **Добавить**.

---

## 11. Комплексная проверка выполнения требований Модуля 1

```bash
# 1. Проверка состояния GRE-туннеля (выполняется на HQ-RTR и BR-RTR)
show interface tunnel.0 // Проверка статуса UP/UP туннельного интерфейса и параметров GRE

# 2. Проверка OSPF-соседства (выполняется на HQ-RTR и BR-RTR)
show ip ospf neighbor // Проверка наличия соседа в статусе Full/Backup или Full/DR

# 3. Проверка сквозной таблицы маршрутизации
show ip route ospf // Проверка присутствия маршрутов удаленного офиса через OSPF

# 4. Проверка трансляций динамического NAT (после отправки тестовых пингов)
show ip nat translations // Просмотр активных трансляций PAT на маршрутизаторах

# 5. Проверка работы DNS-сервера (выполняется на клиенте HQ-CLI)
host hq-srv.au-team.irpo // Прямой DNS-запрос A-записи сервера
host 10.10.100.1 // Обратный DNS-запрос PTR-записи маршрутизатора HQ-RTR
host docker.au-team.irpo // Разрешение имени публичного интерфейса ISP в IP

# 6. Проверка сквозной доступности и доступа в Internet (с клиентских машин)
ping -c 4 10.20.20.2 // Проверка связности между офисами HQ и BR через GRE-туннель
ping -c 4 77.88.8.8 // Проверка выхода узлов во внешнюю сеть Internet через NAT
```
