[BR-SRV]
```bash
# Samba DC: проверка и создание групп/пользователей
samba-tool domain info 127.0.0.1
samba-tool group list
samba-tool user list
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

samba-tool group listmembers hq
samba-tool group listmembers br

# Chrony
chronyc sources
vim /etc/chrony.conf
systemctl restart chronyd
chronyc sources

# Ansible
ansible all -m ping
vim /etc/ansible/hosts
ansible all -m ping
```

[HQ-SRV]
```bash
# Проверка дисков и NFS экспортов
lsblk
blkid
vim /etc/fstab
vim /etc/exports

# Chrony
chronyc sources

# Веб-приложение и база данных
ls /var/www/html
cd /var/www/html/
mount /dev/sr0 /mnt
cp /mnt/web/index.php /var/www/html/
cp /mnt/web/logo.png /var/www/html/
vim /var/www/html/index.php

mariadb -u root
# В MariaDB:
# CREATE USER 'webc'@'localhost' IDENTIFIED BY 'P@ssword';
# GRANT ALL PRIVILEGES ON webdb.* TO 'webc'@'localhost' WITH GRANT OPTION;
# EXIT;

mariadb -u webc -p -D webdb < /mnt/web/dump.sql
systemctl restart mariadb
systemctl restart httpd2
```

[ISP]
```bash
# Chrony
vim /etc/chrony.conf

# Nginx reverse proxy
vim /etc/nginx/sites-enabled.d/reverse-proxy.conf

# Web auth (htpasswd)
htpasswd -c /etc/nginx/.htpasswd WEB
apt-get update
apt-get install apache2-htpasswd -y
htpasswd -c /etc/nginx/.htpasswd WEB
```

[BR-RTR]
```text
en
show ntp status
conf t
ntp server 172.16.1.1
end
write memory
show ntp status
show running-config
```

[HQ-RTR]
```text
en
show ntp status
conf t
ntp server 172.16.1.1
end
write memory
show ntp status
show running-config
```

[BR-FW]
```text
# Команды в документе отсутствуют
```

[HQ-CLI]
```bash
# Chrony
chronyc sources
su
vim /etc/chrony.conf
systemctl restart chronyd
chronyc sources

# Sudo
sudo id
sudo cat
sudo grep
roleadd hq wheel
vim /etc/sudoers
reboot

# NFS
df -h
```

[BR-CLI]
```bash
# Chrony
chronyc sources
su
vim /etc/chrony.conf
systemctl restart chronyd
chronyc sources

# Проверка сессии
w

# NFS
df -h
mount -av
```
