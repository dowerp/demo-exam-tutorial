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
mdadm --create --verbose /dev/md0 –l 0 -n 2 /dev/vdb /dev/vdc
```
Создаем файловую систему на созданном RAID-массиве
```bash
mkfs.ext4 /dev/md0
echo "/dev/md0 /raid ext4 defaults 0 0" >> /etc/fstab
```
Создаем новую директорию
```bash
mkdir -p /raid
```
Применяем монтирования
```bash
mount -av
```
Проверяем
```bash
lsblk
```
<img width="755" height="247" alt="image" src="https://github.com/user-attachments/assets/47c0f0b4-f671-401c-842d-72caaa689805" />

```bash
blkid
```

<img width="758" height="62" alt="image" src="https://github.com/user-attachments/assets/1f692276-e353-45aa-80b5-4646b237e435" />

```bash
df -h
```
<img width="516" height="217" alt="image" src="https://github.com/user-attachments/assets/f9226493-1ac2-42f3-8519-32d3fd77e84a" />

## Настройка сетевой файловой системы 
### Установка
```bash
apt-get install –y nfs-server nfs-utils
```
### Создаем директорию для реализации общего доступа
```bash
mkdir /raid/nfs
```
Выдаем права
```bash
chmod 777 /raid/nfs
```

### Запускаем сервер
```bash
systemctl enable --now nfs-server
```
Пока что переходим к [HQ-CLI](https://github.com/dowerp/demo-exam-tutorial/blob/main/Module-2/HQ-CLI.md#%D0%BF%D1%80%D0%B8%D1%88%D0%BB%D0%B8-%D1%81%D1%8E%D0%B4%D0%B0-%D0%BE%D1%82-hq-srv-%D0%B8-%D0%BF%D0%BE%D0%BA%D0%B0-%D1%82%D1%83%D1%82-%D0%BF%D1%80%D0%BE%D0%B4%D0%BE%D0%BB%D0%B6%D0%B0%D0%B5%D0%BC)

