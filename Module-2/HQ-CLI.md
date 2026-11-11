# Настройка HQ-CLI

## Введение в домен HQ-CLI

### Установка SSSD
```bash
apt-get update && apt-get install -y task-auth-ad-sssd
```
Нажимаем внизу справа на подключения правой кнопкой -> Проводное подключение 1 -> снизу шестерёнка -> параметры IPv4 -> меняем на "Вручную" и там вводим со скрина
<img width="955" height="647" alt="image" src="https://github.com/user-attachments/assets/8b663ba3-b193-4c1c-8109-3b1fc953e7e9" />

Заходим во ВСЕ ПРИЛОЖЕНИЯ и ищем ЦЕНТР УПРАВЛЕНИЯ СИСТЕМОЙ, там переходим в АУТЕНТИФИКАЦИЯ, выбираем Active Directory через SSSD

<img width="675" height="807" alt="image" src="https://github.com/user-attachments/assets/994c21b9-d109-4ab8-a853-e21872b3ffd6" />


Нажимаем применить и вводим пароль домены, который вводили при настройки Samba DC

### Установка библиотеки libnss-role для NSS
```bash
apt-get install -y libnss-role
```
проверяем состояние, должен быть enabled
```bash
control libnss-role
```

Связываем доменную группу hq с локальной wheel
```bash
roleadd hq wheel
```

Раскомментировать WHEEL
```bash
vim +120 /etc/sudoers
```

Дать права в том же файл перед Cmnd_Alias  PROCESSES
```bash
Cmnd_Alias  SHELLCMD = /bin/cat, /bin/grep, /usr/bin/id
```
### Чтобы проверить надо выйти из актуальной учетки и зайти в HQ-CLI
