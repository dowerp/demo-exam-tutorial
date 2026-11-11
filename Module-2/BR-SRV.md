# Настройка BR-SRV

## Настройка контроллера домена Samba-DC

### Установка
```bash
apt-get update && apt-get install -y task-samba-dc && apt-get install -y bind-utils
```


### Развертывание домена
```bash
samba-tool domain provision
```
<details>
  <summary>Если настройка домена выполняется не сразу после установки ОС, перед установкой
необходимо остановить конфликтующие службы krb5kdc и slapd, а также bind</summary>

  ```bash
for service in smb nmb krb5kdc slapd bind; do \
systemctl disable $service; \
systemctl stop $service; \
done
rm -f /etc/samba/smb.conf
rm -rf /var/lib/samba
rm -rf /var/cache/samba
mkdir -p /var/lib/samba/sysvol
```

</details>
<img width="765" height="153" alt="image" src="https://github.com/user-attachments/assets/cabb28d1-ad0d-473b-b315-3aebeb95278f" />

#### Везде кроме пароля жмем энтер

После создания домена необходимо внести изменения в файл /etc/krb5.conf. В этом файле
следует раскомментировать строку default_realm и содержимое разделов realms и
domain_realm, указать название домена (обратите внимание на регистр символов), в строке
dns_lookup_realm должно быть установлено значение false. В момент создания домена Samba
конфигурирует шаблон файла krb5.conf для домена в каталоге /var/lib/samba/private/. Можно
просто заменить этим файлом файл, находящийся в каталоге /etc/:

```bash
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```
Включаем службу samba
```bash
systemctl enable --now samba
```
### Проверяем
Общая инфо о домене и службах
```bash
samba-tool domain info 127.0.0.1
```
<img width="703" height="309" alt="image" src="https://github.com/user-attachments/assets/380ac0b7-15d8-435a-b086-7fe0d68b8469" />

```bash
cat /etc/resolv.conf
```
<img width="753" height="295" alt="image" src="https://github.com/user-attachments/assets/4d2f06d3-e6dd-4807-bd57-4d1f34d8e140" />

Проверяем конфигурации DNS
```bash
host au-team.irpo
```

```bash
host -t SRV _kerberos._udp.au-team.irpo.
```

```bash
host -t SRV _ldap._tcp.au-team.irpo.
```

```bash
host br-srv.au-team.irpo.
```

<img width="546" height="145" alt="image" src="https://github.com/user-attachments/assets/115d7f7b-afc5-4194-bac2-bc5a665bfd9d" />

## Настройка пользовательской структуры домена
### Создание групп и пользователей
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
### Установления неограниченного срока для учетки 
```bash
samba-tool user setexpiry hquser1 --noexpiry
```
### Добавление пользователя
```bash
samba-tool group addmembers «hq» hquser1
```

### Проверка
```bash
samba-tool group list | grep hq
samba-tool group list | grep br
# или
samba-tool group show hq
samba-tool group show br
```

## Введение в домен HQ-CLI

### Установка SSSD
```bash
apt-get update && apt-get install -y task-auth-ad-sssd
```

Заходим во ВСЕ ПРИЛОЖЕНИЯ ищем 


