# Запускаем программы Windows на удаленном Linux сервере

Была у меня как то проблемка, везде используется linux но вот одна программа
работала только на Windows а именно Litemanager NoIP сервер, держать
  ради нее отдельную Windows машину не хотелось, да и с у четом того что от программы
   нужно блыо только чтобы она "проксировала" было решено перенести это на маленькую 
  ubuntu машину где нибудь в облаке
### Подготовка
в данном примере я использую Ubuntu 16.04
#### Устанавливаем Wine

`sudo dpkg --add-architecture i386`
 
`wget -nc https://dl.winehq.org/wine-builds/winehq.key`

`sudo apt-key add winehq.key`

`sudo apt update`

`sudo apt-get install wine`

#### Устанавливаем fluxbox и vnc4server:
`sudo apt-get install fluxbox vnc4server`


### Добавляем пользователя
 
 добавляем пользователя vnc_server 
 
`sudo adduser vnc_server`

переключаемся напользователя vnc_server

`sudo su - vnc_server`

Устанавливаем пароль

`vncpasswd`

Настраиваем VNC

```
vncserver -geometry 800x600 -depth 24
vncserver -kill :1
```

Настраиваем VNC для инеграции с fluxbox

`echo "fluxbox &" >> ~/.vnc/xstartup`

Выходим из vnc_server

`exit`

***
#### Добавляем сервис

`sudo nano /etc/systemd/system/vnc_server.service`

Копируем в этот файл:

```
[Unit]
Description=VNC Server
After=network.target

[Service]
Type=oneshot

ExecStart=/bin/sh -c "/usr/bin/vnc_server.sh start" &>/dev/null &
ExecStop=/bin/sh -c "/usr/bin/vnc_server.sh stop"

RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Сохраняем, перезапускаем systemctl

`sudo systemctl daemon-reload`

Создаем скрипт запуска vncserver имени пользователя vnc_server 

`sudo nano /usr/bin/vnc_server.sh`

```
#!/bin/sh
export PATH="/usr/local/sbin:/sbin:/bin:/usr/sbin:/usr/bin"
#service=vnc_server.service

start() {
    # start daemon
    echo -n "Starting VNC: "
    su -c "vncserver :1 -geometry 800x600 -depth 24" vnc_server
    RETVAL=$?
    return $RETVAL
}

stop() {
    # stop daemon
    echo -n "Stopping VNC: "
    su -c "vncserver -kill :1 >/dev/null 2>&1" vnc_server
    RETVAL=$?
    return $RETVAL
}

case "$1" in
    start)
    start
    RETVAL=$?
    ;;
    stop)
    stop
    RETVAL=$?
    ;;
    restart)
    stop
    sleep 1
    start
    RETVAL=$?
    ;;
    condrestart)
    stop
    sleep 1
    start
    ;;
    status)
    status $service
    RETVAL=0
    ;;
    *)
    echo "Usage: $service {start|stop|restart|condrestart|status}"
    RETVAL=1
    ;;
esac

exit $RETVAL
```

Включаем серис при запуске:

`sudo systemctl enable vnc_server`

Делаем файл запускаемый:

`sudo chmod 775 /usr/bin/vnc_server.sh`

Перезагружаемся:

`sudo reboot`


### Копируем программу на сервер

`sudo mkdir  ~/lm`

с машины пользователя

`scp -r ~/lm пользовательssh@адрессервера:~/lm`

### Переносим в папку vnc_server
```
sudo mkdir /home/vnc_server/lm & sudo mv ~/lm/* /home/vnc_server/lm 
sudo chown vnc_server /home/vnc_server/lm/*
```


### Подключаемся по VNC:

Используя люой vnc клиент подключаемся к серверу 
правой кнопкой мыши запускаем Applications>Terminal Emulators>Xterm
и в водим 

`wine ~/lm/LMNoIpServer.exe`

программа запущена.
### Автозапуск программы при старте системы:

`sudo nano /home/vnc_server/.fluxbox/startup`

и перед exec fluxbox добавляем 

`wine ~/lm/LMNoIpServer.exe`





хочу так же добавить что LMNoIpServer была выбрана в качестве примера
