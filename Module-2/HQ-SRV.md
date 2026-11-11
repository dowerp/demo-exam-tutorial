# Настройка HQ-SRV

## Конфигурация файлового хранилища
### Просматриваем все подключенные устройства
```bash
lsblk
```
<img width="759" height="148" alt="image" src="https://github.com/user-attachments/assets/7565ed15-6084-4e46-9b91-41a0977df41d" />

### Устанавливаем пакет mdadm для создания raid массива
```bash
apt-get install -y mdadm
```
### Создание RAID-массива с mdadm
```bash
mdadm --create --verbose /devmd0 –l 0 -n 2 /dev/sdb /dev/sdc
```
