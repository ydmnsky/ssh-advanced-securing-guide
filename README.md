# Продвинутая настройка безопасности для подключения по SSH к удалённому серверу.

## Обновляем пакеты
```console
sudo apt-get update && sudo apt-get upgrade
```
## Устанавливаем демон knockd
```console
sudo apt install knockd
```
## Редактируем файл /etc/knockd.conf
```
[options]
        logfile = /var/log/knockd.log

[openSSH]
        sequence    = <Последовательность портов для открытия>
        seq_timeout = 5
        command     = systemctl start ssh.service
        tcpflags    = syn

[closeSSH]
        sequence    = <Последовательность портов для закрытия>
        seq_timeout = 5
        command     = systemctl stop ssh.service
        tcpflags    = syn
```


## Включаем его и добавляем в автозагрузку
```console
sudo systemctl start knockd.service
sudo systemctl enable knockd.service
```
Отключаемся от удаленного сервера командой exit или сочетанием клавиш Ctrl+A+D
## Создаем скрипт для автоматического подключения на локальной машине autoknock.sh
```
#!/bin/bash

knock <IP-адрес сервера> <Последовательность портов для открытия>
sleep 5

ssh <Имя юзера>@<IP-адрес сервера>

knock <IP-адрес сервера> <Последовательность портов для закрытия>
```
## Делаем его исполняемым
```console
chmod +x autoknock.sh
```
## Подключаемся к серверу
./autoknock.sh


## Устанавливаем fail2ban
```console
sudo apt install fail2ban
```
## Cоздаем jail для банов IP-адресов после неудачных попыток подключений в файле /etc/fail2ban/jail.local
```
[sshd]
enabled = true
mode = aggressive
port = ssh
logpath = /var/log/auth.log
maxretry = 3
findtime = 600
bantime = 24h
ignoreip = 127.0.0.1

```
## Запускаем сервис fail2ban и добавляем его в автозапуск
```console
sudo systemctl start knockd.service
sudo systemctl enable knockd.service
```

## Чтобы посмотреть забаненные IP-адреса
```console
sudo fail2ban-client status sshd
```

