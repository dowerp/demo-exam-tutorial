# Модуль 2: Настройка сетевых сервисов, централизованной аутентификации и приложений

В данном руководстве исключены избыточные и повторяющиеся проверочные команды. Для всех конфигурационных файлов приведены точные параметры для внесения или исправления с построчным разбором директив.

---

## [BR-SRV] — Контроллер домена и сервер автоматизации филиала

### 1. Настройка контроллера домена Samba DC (Группы и пользователи)
Создание доменных групп и пользователей через встроенную утилиту `samba-tool`:

```bash
samba-tool group add hq
samba-tool group add br

for i in {1..5}; do
  samba-tool user add hquser$i P@ssw0rd
  samba-tool user setexpiry hquser$i --noexpiry
  samba-tool group addmembers "hq" hquser$i
done

for i in {1..5}; do
  samba-tool user add bruser$i P@ssw0rd
  samba-tool user setexpiry bruser$i --noexpiry
  samba-tool group addmembers "br" bruser$i
done
```

* **Разбор команд и параметров:**
  * `samba-tool group add <имя>` — создание глобальной группы безопасности Active Directory.
  * `for i in {1..5}; do ... done` — bash-цикл для пакетного создания пяти пользователей с $1$ по $5$.
  * `samba-tool user add hquser$i P@ssw0rd` — создание учетной записи с начальным паролем `P@ssw0rd`.
  * `samba-tool user setexpiry hquser$i --noexpiry` — отключение срока действия пароля (учетная запись не потребует смены пароля при первом входе).
  * `samba-tool group addmembers "<группа>" <пользователь>` — добавление учетной записи в состав доменной группы.

---

### 2. Синхронизация времени (Chrony)
Откройте файл конфигурации NTP-клиента:
```bash
vim /etc/chrony.conf
```

**Что изменить:**
Найти строку с директивой `pool` или `server` и заменить на адрес шлюза провайдера ISP:
```ini
pool 172.16.1.1 iburst
```

* **Разбор директив:**
  * `pool 172.16.1.1` — указание источника синхронизации времени (внешний NTP-сервер на машине ISP).
  * `iburst` — режим ускоренной начальной синхронизации: при старте службы отправляется пачка из первых 4–8 запросов для мгновенного захвата точного времени.

**Перезапуск службы времени:**
```bash
systemctl restart chronyd
```

---

### 3. Настройка инвентаря Ansible
Откройте файл инвентаря хостов:
```bash
vim /etc/ansible/hosts
```

**Что внести (полная замена или добавление групп `[hq]` и `[br]`):**
```ini
[hq]
hq-srv ansible_host=10.10.100.2 ansible_port=2026 ansible_ssh_user=root ansible_ssh_pass=toor
hq-cli ansible_host=10.10.200.2 ansible_port=22 ansible_ssh_user=root ansible_ssh_pass=toor
hq-rtr ansible_host=172.16.1.2 ansible_port=22 ansible_ssh_user=net_admin ansible_ssh_pass=P@ssword ansible_connection=network_cli ansible_network_os=ios

[br]
br-rtr ansible_host=172.16.2.2 ansible_port=22 ansible_ssh_user=net_admin ansible_ssh_pass=P@ssword ansible_connection=network_cli ansible_network_os=ios
br-cli ansible_host=10.20.30.2 ansible_port=22 ansible_ssh_user=root ansible_ssh_pass=toor
```

* **Разбор переменных Ansible:**
  * `[hq]`, `[br]` — логические группы хостов.
  * `ansible_host` — целевой IPv4-адрес устройства для подключения по SSH.
  * `ansible_port` — порт SSH-демона (для `hq-srv` используется нестандартный проброшенный порт $2026$).
  * `ansible_ssh_user` / `ansible_ssh_pass` — учетные данные для авторизации без предварительно настроенных SSH-ключей.
  * `ansible_connection=network_cli` — специализированный плагин подключения для сетевых операционных систем (вместо стандартной оболочки Linux).
  * `ansible_network_os=ios` — синтаксический профиль синтаксиса Cisco IOS / EcoRouter CLI.

**Проверка доступности хостов:**
```bash
ansible all -m ping
```

---
---

## [HQ-SRV] — Сервер центрального офиса (Веб-сервер и СУБД)

### 1. Подготовка файлов веб-приложения
Монтирование диска с дополнительными материалами и перенос веб-компонентов:
```bash
cd /var/www/html/
mount /dev/sr0 /mnt
cp /mnt/web/index.php /var/www/html/
cp /mnt/web/logo.png /var/www/html/
```

* **Разбор команд:**
  * `mount /dev/sr0 /mnt` — монтирование виртуального оптического привода с исходными файлами в директорию `/mnt`.
  * `cp /mnt/web/... /var/www/html/` — копирование исполняемого PHP-скрипта и графического логотипа в корневую директорию Apache HTTPD.

---

### 2. Настройка подключения к базе данных
Откройте файл скрипта веб-приложения:
```bash
vim /var/www/html/index.php
```

**Что исправить (найти блок параметров подключения `$servername`, `$username` и т.д.):**
```php
$servername = "localhost";
$username = "webc";
$password = "P@ssword";
$dbname = "webdb";
```

* **Разбор параметров:**
  * `$servername = "localhost"` — адрес сервера СУБД MariaDB (локальный сокет).
  * `$username = "webc"` — имя выделенного пользователя БД.
  * `$password = "P@ssword"` — пароль пользователя `webc`.
  * `$dbname = "webdb"` — имя целевой рабочей базы данных приложения.

---

### 3. Настройка СУБД MariaDB и импорт дампа
Создание пользователя и назначение прав внутри базы данных:
```bash
mariadb -u root
```

**В интерактивной консоли MariaDB выполнить:**
```sql
CREATE USER 'webc'@'localhost' IDENTIFIED BY 'P@ssword';
GRANT ALL PRIVILEGES ON webdb.* TO 'webc'@'localhost' WITH GRANT OPTION;
EXIT;
```

* **Разбор SQL-запросов:**
  * `CREATE USER 'webc'@'localhost' IDENTIFIED BY 'P@ssword';` — создание учетной записи с ограничением входа только с локального хоста.
  * `GRANT ALL PRIVILEGES ON webdb.* ...` — выдача полных прав на чтение, запись и изменение всех таблиц в схеме `webdb`.
  * `WITH GRANT OPTION` — право передавать выданные полномочия другим пользователям.

**Импорт схемы и данных из дампа:**
```bash
mariadb -u webc -p -D webdb < /mnt/web/dump.sql
```
*(При запросе пароля ввести: `P@ssword`)*

* **Разбор команды:**
  * `-u webc` — авторизация под созданным пользователем.
  * `-p` — интерактивный запрос пароля.
  * `-D webdb` — выбор целевой базы данных перед выполнением SQL-команд.
  * `< /mnt/web/dump.sql` — перенаправление входного потока из исходного файла дампа.

**Перезапуск веб-сервера и СУБД:**
```bash
systemctl restart mariadb
systemctl restart httpd2
```

---
---

## [ISP] — Маршрутизатор Интернет-провайдера (Nginx Reverse Proxy & Auth)

### 1. Установка утилиты генерации паролей Nginx и создание пользователя
```bash
apt-get update
apt-get install apache2-htpasswd -y
htpasswd -c /etc/nginx/.htpasswd WEB
```
*(Дважды ввести пароль: `P@ssword`)*

* **Разбор команд и параметров:**
  * `apt-get install apache2-htpasswd -y` — установка пакета, содержащего бинарный файл `htpasswd` для генерации хэшей учетных данных HTTP Basic Auth.
  * `htpasswd -c /etc/nginx/.htpasswd WEB`:
    * `-c` (create) — создание нового файла паролей (перезаписывает старый при наличии);
    * `/etc/nginx/.htpasswd` — путь к файлу учетных записей, на который ссылается директива `auth_basic_user_file` в конфигурации Nginx;
    * `WEB` — имя создаваемого пользователя для входа на веб-ресурс.

---
---

## [HQ-RTR] — Маршрутизатор центрального офиса (EcoRouter)

### 1. Настройка синхронизации времени по протоколу NTP
```text
en
conf t
ntp server 172.16.1.1
end
write memory
```

* **Разбор команд:**
  * `ntp server 172.16.1.1` — добавление IP-адреса маршрутизатора ISP в качестве вышестоящего источника синхронизации времени Stratum 5.
  * `write memory` — персистентное сохранение конфигурации в памяти устройства.

---
---

## [BR-RTR] — Маршрутизатор филиала (EcoRouter)

### 1. Настройка синхронизации времени по протоколу NTP
```text
en
conf t
ntp server 172.16.1.1
end
write memory
```
* **Разбор команд:** Указание центрального сервера точного времени ISP ($172.16.1.1$) и сохранение настроек.

---
---

## [HQ-CLI] — Клиентский ПК центрального офиса

### 1. Синхронизация времени (Chrony)
Перейдите под учетную запись суперпользователя:
```bash
su -
```
*(Пароль: `toor`)*

Откройте конфигурационный файл:
```bash
vim /etc/chrony.conf
```

**Что изменить:**
```ini
pool 172.16.1.1 iburst
```

**Применение:**
```bash
systemctl restart chronyd
```

---

### 2. Ввод рабочей станции в домен Active Directory
Выполняется через графический интерфейс «Центр управления системой» (Alterator):
1. Открыть: **Меню** $\rightarrow$ **Система** $\rightarrow$ **Центр управления системой** (ввести пароль `toor`).
2. Перейти в раздел: **Пользователи** $\rightarrow$ **Аутентификация**.
3. Выбрать режим доменной аутентификации, указать домен `AU-TEAM.IRPO`.
4. Нажать кнопку **«Применить»**.
5. Во всплывающем окне ввести учетные данные администратора домена:
   * **Имя пользователя**: `Administrator`
   * **Пароль**: `P@ssword`
6. Подтвердить присоединение к домену (`Добро пожаловать в домен AU-TEAM.IRPO`).

---

### 3. Ограничение прав sudo для группы доменных пользователей
Переключитесь во второй виртуальный терминал комбинацией клавиш `Ctrl+Alt+F2`, выполните вход под `root` (пароль `toor`) и выполните команды:

```bash
roleadd hq wheel
```

* **Разбор:** Привязка доменной группы `hq` к локальной системной роли `wheel` в ALT Linux через подсистему ролевого доступа tcb.

Откройте файл правил `sudo`:
```bash
vim /etc/sudoers
```

**Что внести в конец файла:**
```text
Cmnd_Alias DE = /bin/cat, /bin/grep, /usr/bin/id
WHEEL_USERS ALL=(ALL:ALL) DE
```

* **Разбор директив:**
  * `Cmnd_Alias DE = /bin/cat, /bin/grep, /usr/bin/id` — создание алиаса команд `DE`, строго ограничивающего перечень разрешенных исполняемых файлов утилитами `cat`, `grep` и `id`.
  * `WHEEL_USERS ALL=(ALL:ALL) DE` — разрешение членам группы `wheel` (и доменной группы `hq`) выполнять исключительно команды из списка `DE` с повышением привилегий.

**Применение настроек безопасности:**
```bash
reboot
```

---
---

## [BR-CLI] — Клиентский ПК филиала

### 1. Синхронизация времени (Chrony)
```bash
su -
```
*(Пароль: `toor`)*

Откройте конфигурационный файл:
```bash
vim /etc/chrony.conf
```

**Что изменить:**
```ini
pool 172.16.1.1 iburst
```

**Применение:**
```bash
systemctl restart chronyd
```

---

### 2. Монтирование сетевой файловой системы NFS
Переключитесь во второй виртуальный терминал клавишами `Ctrl+Alt+F2` и авторизуйтесь под `root` (пароль `toor`):

```bash
mount -av
```

* **Разбор команды и параметров:**
  * `mount -a` — автоматическое монтирование всех файловых систем, описанных в конфигурационном файле `/etc/fstab` (за исключением помеченных опцией `noauto`).
  * `-v` (verbose) — подробный вывод статуса подключения, подтверждающий монтирование удаленного каталога `10.10.100.2:/raid/nfs` в локальную точку `/mnt/nfs`.
