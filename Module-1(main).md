# Справочник команд для решения Модуля 1 

## 1. ISP 

### 1.1. Настройка имени узла
```bash
hostnamectl set-hostname isp.au-team.irpo // Установка полного доменного имени узла
```

### 1.2. Проверка интерфейсов и маршрутизации
```bash
ip a // Проверка адресов (ens3 - DHCP, ens4 - 172.16.1.1/28, ens5 - 172.16.2.1/28; преднастроено)
cat /etc/net/sysctl.conf | grep ip_forward // Проверка включения пересылки пакетов (net.ipv4.ip_forward = 1)
```

### 1.3. Настройка трансляции адресов (NAT/PAT)
```bash
iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE // Включение маскарадинга (PAT) на исходящем интерфейсе ens3
iptables-save >> /etc/sysconfig/iptables // Сохранение текущих правил iptables в файл конфигурации
systemctl enable --now iptables // Активация автозапуска и немедленный старт службы iptables
iptables -t nat -L // Проверка наличия правила MASQUERADE в цепочке POSTROUTING
```

### 1.4. Проверка часового пояса
```bash
timedatectl // Проверка часового пояса (Europe/Moscow, UTC+3)
```

---

## 2. HQ-RTR 

### 2.1. Имя узла и домен
```bash
enable // Вход в привилегированный режим EXEC
configure terminal // Переход в режим глобальной конфигурации
hostname HQ-RTR // Установка сетевого имени маршрутизатора
ip domain-name au-team.irpo // Установка локального доменного суффикса
```

### 2.2. Статический маршрут по умолчанию
```bash
ip route 0.0.0.0 0.0.0.0 172.16.1.1 // Создание шлюза по умолчанию через IP-адрес провайдера ISP
```

### 2.3. Настройка туннеля GRE
```bash
interface tunnel.0 // Создание логического туннельного интерфейса tunnel.0
ip address 10.10.10.1/30 // Назначение IP-адреса туннеля со стороны HQ
ip tunnel 172.16.1.2 172.16.2.2 mode gre // Настройка инкапсуляции GRE с внешнего адреса HQ на внешний адрес BR
exit // Выход из режима настройки туннельного интерфейса
```

### 2.4. Динамическая маршрутизация OSPF
```bash
router ospf 0 // Запуск процесса динамической маршрутизации OSPF с идентификатором 0
network 10.10.100.0/27 area 0 // Анонс подсети серверов vl100 в магистральную область 0
network 10.10.200.0/28 area 0 // Анонс подсети клиентов vl200 в магистральную область 0
network 10.10.30.0/29 area 0 // Анонс подсети управления vl999 в магистральную область 0
network 10.10.10.0/30 area 0 // Анонс подсети GRE-туннеля в магистральную область 0
passive-interface default // Перевод всех интерфейсов в пассивный режим по умолчанию
no passive-interface tunnel.0 // Включение отправки сообщений OSPF Hello через GRE-туннель
area 0 authentication // Активация аутентификации пакетов OSPF в области 0
exit // Выход из конфигурации процесса OSPF
interface tunnel.0 // Переход к настройке туннельного интерфейса
ip ospf authentication message-digest // Включение криптографической аутентификации MD5 на интерфейсе
ip ospf message-digest-key 1 md5 P@ssword // Задание ключа аутентификации №1 с паролем P@ssword
exit // Выход из интерфейса tunnel.0
```

### 2.5. Динамическая трансляция адресов (NAT)
```bash
interface isp // Переход к интерфейсу подключения к ISP
ip nat outside // Назначение интерфейса внешней зоной NAT
exit // Выход из интерфейса isp
interface vl100 // Переход к интерфейсу серверов
ip nat inside // Назначение интерфейса внутренней зоной NAT
exit // Выход из интерфейса vl100
interface vl200 // Переход к интерфейсу клиентов
ip nat inside // Назначение интерфейса внутренней зоной NAT
exit // Выход из интерфейса vl200
interface vl999 // Переход к интерфейсу управления
ip nat inside // Назначение интерфейса внутренней зоной NAT
exit // Выход из интерфейса vl999
ip nat pool HQ 10.10.0.1-10.10.200.254 // Создание пула локальных адресов HQ для трансляции
ip nat source dynamic inside-to-outside pool HQ overload interface isp // Включение динамического PAT пула HQ через адрес порта isp
```

### 2.6. Часовой пояс и сохранение
```bash
ntp timezone utc+3 // Установка системного часового пояса UTC+3 (Москва)
end // Выход в привилегированный режим EXEC
write memory // Запись текущей конфигурации в энергонезависимую память
```

### 2.7. Проверочные команды на HQ-RTR
```bash
show ip interface brief // Проверка состояния сетевых интерфейсов
show interface tunnel.0 // Проверка состояния GRE-туннеля
show ip route // Проверка таблицы маршрутизации (наличие S* и OSPF)
show ip nat translations // Просмотр таблицы активных трансляций NAT
show dhcp-server 1 detailed // Проверка параметров встроенного сервера DHCP
```

---

## 3. BR-RTR 

### 3.1. Имя узла и домен
```bash
enable // Вход в привилегированный режим EXEC
configure terminal // Переход в режим глобальной конфигурации
hostname BR-RTR // Установка сетевого имени маршрутизатора
ip domain-name au-team.irpo // Задание доменного имени
```

### 3.2. Статический маршрут по умолчанию
```bash
ip route 0.0.0.0 0.0.0.0 172.16.2.1 // Создание маршрута по умолчанию через ISP (172.16.2.1)
```

### 3.3. Настройка туннеля GRE
```bash
interface tunnel.0 // Создание логического туннельного интерфейса tunnel.0
ip address 10.10.10.2/30 // Назначение IP-адреса туннеля со стороны филиала BR
ip tunnel 172.16.2.2 172.16.1.2 mode gre // Настройка инкапсуляции GRE с внешнего адреса BR на внешний адрес HQ
exit // Выход из режима интерфейса tunnel.0
```

### 3.4. Динамическая маршрутизация OSPF
```bash
router ospf 0 // Запуск процесса OSPF с идентификатором 0
network 10.20.10.0/30 area 0 // Анонс подсети стыка с межсетевым экраном BR-FW
network 10.10.10.0/30 area 0 // Анонс подсети GRE-туннеля в область 0
passive-interface default // Блокировка рассылки Hello на всех интерфейсах
no passive-interface tunnel.0 // Разрешение обмена OSPF через GRE-туннель
no passive-interface fw // Разрешение установления OSPF-соседства с BR-FW
area 0 authentication // Включение проверки подлинности OSPF в области 0
exit // Выход из режима OSPF
interface tunnel.0 // Переход в туннельный интерфейс
ip ospf authentication message-digest // Включение аутентификации MD5 на туннеле
ip ospf message-digest-key 1 md5 P@ssword // Установка ключа MD5 №1 с паролем P@ssword
exit // Выход из интерфейса tunnel.0
interface fw // Переход в интерфейс связи с BR-FW
ip ospf authentication message-digest // Включение аутентификации MD5 на линке к файрволу
ip ospf message-digest-key 1 md5 P@ssword // Установка ключа MD5 №1 с паролем P@ssword
exit // Выход из интерфейса fw
```

### 3.5. Динамическая трансляция адресов (NAT)
```bash
interface isp // Переход к внешнему интерфейсу подключения к ISP
ip nat outside // Назначение интерфейса внешней зоной NAT
exit // Выход из интерфейса isp
interface fw // Переход к интерфейсу подключения в сторону филиала BR
ip nat inside // Назначение интерфейса внутренней зоной NAT
exit // Выход из интерфейса fw
ip nat pool BR 10.0.0.1-10.254.254.254 // Определение пула адресов внутренних подсетей филиала
ip nat source dynamic inside-to-outside pool BR overload interface isp // Включение PAT пула сетей BR в адрес порта isp
```

### 3.6. Часовой пояс и сохранение
```bash
ntp timezone utc+3 // Установка системного часового пояса UTC+3 (Москва)
end // Возврат в привилегированный режим
write memory // Сохранение конфигурации в память
```

### 3.7. Проверочные команды на BR-RTR
```bash
show ip ospf neighbor // Проверка установления соседства по OSPF (статус Full)
show ip route // Проверка маршрутов OSPF и маршрута по умолчанию
show ip nat translations // Проверка таблицы динамических трансляций NAT
```

---

## 4. HQ-SW 

```bash
hostnamectl set-hostname hq-sw.au-team.irpo // Установка имени узла коммутатора
ovs-vsctl show // Проверка конфигурации моста hq-sw (порты ens3 trunk, ens4 tag 100, ens5 tag 200, MGMT tag 999 преднастроены)
```

---

## 5. HQ-SRV 

### 5.1. Имя узла
```bash
hostnamectl set-hostname hq-srv.au-team.irpo // Установка полного доменного имени сервера
```

### 5.2. Настройка сетевого адаптера ens3
```bash
echo "10.10.100.2/27" > /etc/net/ifaces/ens3/ipv4address // Назначение статического IPv4-адреса и маски подсети
echo "default via 10.10.100.1" > /etc/net/ifaces/ens3/ipv4route // Установка маршрута по умолчанию через HQ-RTR
cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf // Запись конфигурации DNS-клиента
search au-team.irpo
nameserver 77.88.8.8
nameserver 127.0.0.1
EOF
systemctl restart network // Применение сетевых настроек перезапуском службы сети
```

### 5.3. Настройка прав sudo для группы wheel
```bash
sed -i 's/^#*WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL/WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL/' /etc/sudoers // Разрешение беспарольного вызова sudo участникам группы wheel
```

### 5.4. Настройка SSH
```bash
sed -i 's|^#*Banner .*|Banner /etc/openssh/banner|' /etc/openssh/sshd_config // Раскомментирование директивы Banner в конфигурации sshd
echo "Authorized access only" > /etc/openssh/banner // Создание текста баннера безопасности
systemctl restart sshd // Перезапуск службы SSH для применения нового баннера
```

### 5.5. Настройка DNS-сервера (BIND)

#### Конфигурация параметров (/etc/bind/options.conf):
Внести директивы в блок `options { ... }`:
```text
listen-on { 10.10.100.2; 127.0.0.1; }; // Адреса прослушивания DNS-запросов
listen-on-v6 { none; }; // Отключение IPv6
forwarders { 77.88.8.8; }; // Внешний DNS-сервер пересылки
allow-query { any; }; // Разрешение запросов от любых клиентов
```

#### Файл прямой зоны (/etc/bind/zone/au-team.irpo.zone):
```text
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

#### Файл обратной зоны (/etc/bind/zone/10.10.in-addr.arpa):
```text
$TTL 1D
@   IN  SOA au-team.irpo. root.au-team.irpo. ( // SOA-запись зоны обратного просмотра
            2025110600 12H 1H 1W 1H ) // Серийный номер и таймеры
    IN  NS  au-team.irpo. // NS-запись зоны

1.100   IN  PTR hq-rtr.au-team.irpo. // PTR-запись адреса 10.10.100.1
2.100   IN  PTR hq-srv.au-team.irpo. // PTR-запись адреса 10.10.100.2
2.200   IN  PTR hq-cli.au-team.irpo. // PTR-запись адреса 10.10.200.2
```

#### Запуск службы DNS:
```bash
systemctl enable --now bind.service // Добавление BIND в автозагрузку и запуск демона named
```

---

## 6. HQ-CLI 
```bash
hostnamectl set-hostname hq-cli.au-team.irpo // Установка имени клиентской станции
ip a // Проверка получения адреса 10.10.200.2/28 по DHCP на ens3
timedatectl // Проверка часового пояса (Europe/Moscow)
```

---

## 7. BR-CLI (Клиент филиала — ALT Workstation)

```bash
hostnamectl set-hostname br-cli.au-team.irpo // Установка имени клиентской станции филиала
su - // Переход в сессию root (пароль: toor)
mv /etc/net/ifaces/ens18 /etc/net/ifaces/ens3 // Переименование каталога адаптера ens18 в ens3
echo "10.20.30.2/29" > /etc/net/ifaces/ens3/ipv4address // Назначение статического адреса рабочей станции BR
echo "default via 10.20.30.1" > /etc/net/ifaces/ens3/ipv4route // Установка маршрута по умолчанию на BR-FW
cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf // Конфигурация резолвера DNS
search au-team.irpo
nameserver 10.10.100.2
EOF
systemctl restart network // Применение сетевых настроек
ip a // Проверка назначенного адреса 10.20.30.2/29 на адаптере ens3
```

---

## 8. BR-SRV 

### 8.1. Имя узла
```bash
hostnamectl set-hostname br-srv.au-team.irpo // Установка имени сервера филиала
```

### 8.2. Сеть
```bash
echo "10.20.20.2/28" > /etc/net/ifaces/ens3/ipv4address // Назначение статического IP-адреса сервера филиала
echo "default via 10.20.20.1" > /etc/net/ifaces/ens3/ipv4route // Установка шлюза по умолчанию на интерфейс BR-FW
cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf // Запись параметров поиска домена и DNS-сервера
search au-team.irpo
nameserver 10.10.100.2
EOF
systemctl restart network // Перезапуск сетевой службы
```

### 8.3. Sudo
```bash
sed -i 's/^#*WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL/WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL/' /etc/sudoers // Разрешение беспарольного sudo группе wheel
```

### 8.4. SSH
```bash
sed -i 's|^#*Banner .*|Banner /etc/openssh/banner|' /etc/openssh/sshd_config // Включение параметра пути баннера в sshd_config
echo "Authorized access only" > /etc/openssh/banner // Создание текста баннера
systemctl restart sshd // Перезапуск службы SSH
```

---

## 9. BR-FW 

### 9.1. Консоль / Терминал Ideco
```bash
hostnamectl set-hostname br-fw.au-team.irpo // Установка имени межсетевого экрана
ip -4 a // Проверка интерфейсов: Leth2 (10.20.10.2/30), Leth3 (10.20.20.1/28), Leth4 (10.20.30.1/29)
```

### 9.2. Веб-интерфейс Ideco NGFW (https://10.20.30.1:8443)
* **Авторизация:** Логин `Admin`, Пароль `IdecoP@ssword`.
* **Настройки OSPF:**
  * Перейти: **Сервисы -> Маршрутизация -> OSPF**.
  * Переключатель OSPF: **Работает** // Включение процесса OSPF
  * Router ID: `10.20.30.1` // Назначение Router ID файрвола
  * Аутентификация соседей: выбрать `MD5` // Тип аутентификации
  * Key ID: `1` // Идентификатор ключа
  * Пароль: `P@ssword` // Пароль ключа аутентификации
  * Нажать кнопку **Сохранить** // Применение глобальных параметров OSPF
* **Добавление интерфейса в область OSPF:**
  * Перейти во вкладку **«Локальные интерфейсы»** -> кнопка **«+ Добавить»**.
  * Интерфейс: `oif0` (стык с BR-RTR / Leth2) // Выбор интерфейса соединения с BR-RTR
  * Область (Area): `0` // Назначение интерфейса в магистральную область OSPF 0
  * Тип области: `Normal` // Стандартный тип области
  * Стоимость: `1` // Стоимость маршрута через интерфейс
  * Нажать кнопку **Добавить** // Активация OSPF на порту
