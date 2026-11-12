# Настройка ISP

## Настройка Chronyd

### Вносим изменения в файл конфигурации
```bash
vim /etc/chrony.conf
```

<img width="758" height="143" alt="image" src="https://github.com/user-attachments/assets/67167cd7-91c4-4a19-b526-4d62c1b0b3bb" />

### В Этом же файле на всех устройствах делаем так

<img width="750" height="84" alt="image" src="https://github.com/user-attachments/assets/9efe1303-f49c-4e69-b500-f17bc1c46569" />


### Перезагружаем Chronyd ВЕЗДЕ!!
```bash
systemctl restart chronyd
```

