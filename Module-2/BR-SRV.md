# Настройка BR-SRV

## Настройка Samba-DC

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


### Ввод в состав домена

### Создание групп

### Обеспечение возможности входа

### Настройка привилегий 

