# Настройка BR-SRV

## Настройка контроллера домена Samba-DC

### Установка
```bash
apt-get update && apt-get install -y task-samba-dc
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

Везде кроме пароля жмем энтер
<details>
<summary> Если не грузит изображение там:</summary>
```bash
[root@br-srv ~]# samba-tool domain provision
Realm [AU-TEAM.IRPO]:
Domain [AU-TEAM]:
Server Role (dc, member, standalone) [dc]:
DNS backend (SAMBA_INTERNAL, BIND9_FLATFILE, BIND9_DLZ, NONE) [SAMBA_INTERNAL]:
DNS forwarder IP address (write 'none' to disable forwarding): 192.168.100.2
Administrator password:
Retype password: 
```
</details>
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
#### Проверяем
Общая инфо о домене и службах
```bash
samba-tool domain info 127.0.0.1
```
<img width="703" height="309" alt="image" src="https://github.com/user-attachments/assets/380ac0b7-15d8-435a-b086-7fe0d68b8469" />

```bash
cat /etc/resolv.conf
```
<img width="753" height="295" alt="image" src="https://github.com/user-attachments/assets/4d2f06d3-e6dd-4807-bd57-4d1f34d8e140" />

