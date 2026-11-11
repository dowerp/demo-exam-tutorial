# Настройка BR-SRV

## Настройка Samba-DC

### Установка
```bash
apt-get update && apt-get install -y task-samba-dc
```
<details>
  <summary>Если настройка домена выполняется не сразу после установки ОС, перед установкой
необходимо остановить конфликтующие службы krb5kdc и slapd, а также bind</summary>

  ```bash
for service in smb nmb krb5kdc slapd bind; do \
systemctl disable $service; \
systemctl stop $service; \
done

```
</details>

### Развертывание домена

### Ввод в состав домена

### Создание групп

### Обеспечение возможности входа

### Настройка привилегий 

