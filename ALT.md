
## Задание demo2024 №1
1.Выполните базовую настройку всех устройств:
a. Соберите топологию согласно рисунку. Все устройства работают на OC Linux - ALT
- ISP - Альт сервер 10.2 (CLI)
- CLI - Альт Рабочая станция 10.2 (GUI)
- HQ-R - Альт сервер 10.2 (CLI)
- HQ-SRV - Альт сервер 10.2 (GUI)
- BR-R - Альт сервер 10.2 (CLI)
- BR-SRV - Альт сервер 10.2 (CLI)

b. Присвоить имена в соответствии с топологией

c. Рассчитайте IP-адресацию IPv4. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.  

d. Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.  

e. Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.  

### Таблица ip адресов
|Имя устройства|Интерфейс|IPv4/IPv6    |Маска/Префикс|Шлюз         |
|--------------|---------|-------------|-------------|-------------|
|ISP           |ens192   |192.168.0.170|/30          |             |
|              |ens224   |192.168.0.162|/30          |             |
|              |ens256   |10.12.32.6   |/24          |             |
|              |ens161   |192.168.0.166|/30          |             |
|CLI           |ens192   |192.168.0.165|/30          |192.168.0.166|
|HQ-R          |ens224   |192.168.0.5  |/25          |             |
|              |ens192   |192.168.0.169|/30          |192.168.0.170|
|BR-R          |ens192   |192.168.0.136|/27          |             |
|              |ens224   |192.168.0.161|/30          |192.168.0.162|
|HQ-SRV        |ens192   |192.168.0.8  |/25          |192.168.0.5  |
|BR-SRV        |ens192   |192.168.0.135|/27          |192.168.0.136|

### Топология
![image](https://github.com/danakahara19/demo2024/assets/148867574/c8389980-703a-40e6-b500-00f9ff21e9c6)

В HR-SRV и CLI в терминале прописывать команду что бы перейти в привилигерованного пользователя 
```
su -
```
Смотрим интерфейсы к Ip адресам
```
ip a
```
Открываем файл options и редактируем как показано   
```
vim /etc/net/ifaces/ens__/options
```
Если файл не сохраняется пистаь команду 
```
mkdir /etc/net/ifaces/ens__
```
и прописывать этот текст самостоятельно
```
BOOTPROTO=static
TYPE=eth
CONFIG_WIRELESS=no
SYSTEMD_BOOTPROTO=dhcp4
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=no
```
Ставим IP адреса и шлюз
```
echo 192.168.0.169/30 > /etc/net/ifaces/ens__/ipv4address
```
```
echo default via 192.168.0.170 > /etc/net/ifaces/ens__/ipv4route
```
> вместо __ писать интерфейс на который ставим ip адрес

После установки IP адресов перезагружаем сеть
```
service network restart 
```
что бы ip не убирались и не было второго 
```
systemctl disable network.service NetworkManager
```
Если нужно указать информацию о DNS-сервере, прописываем команду:
```
echo nameserver 8.8.8.8 > /etc/resolv.conf
```
### Настройка NAT на ISP HQ-R BR-R
установка пакетов Firewalld
```
apt-get -y install firewall
```
автозагрузка
```     
systemctl enable --now firewalld
```
Правила к исходяшим пакетам
```
firewall-cmd --permanent --zone=public --add-interface=ens__
```
>public интерфейс смотрящий в сторону интернета
Правила к входяшим пакетам
```
firewall-cmd --permanent --zone=trusted --add-interface=ens34
```
>trusted адреса внутренней локальной сети
Включаем NAT
```
firewall-cmd --permanent --zone=public --add-masquerade
```
Сохраняем
```
firewall-cmd --reload
```
#### Настройка для того что бы тунннель и FRR работали с NAT
Включаем пересылку всех пакетов на **ISP** между BR-R и HQ-R
```
firewall-cmd --direct --permanent --add-rule ipv4 filter FORWARD 0 -i ens224 -o ens192 -j ACCEPT
firewall-cmd --direct --permanent --add-rule ipv4 filter FORWARD 0 -i ens192 -o ens224 -j ACCEPT
```
Открываем порты OSPF на ISP:
```
firewall-cmd --permanent --zone=trusted --add-port=89/tcp
firewall-cmd --permanent --zone=trusted --add-port=89/udp
```
**HQ-R и BR-R** Включаем пересылку между интерфейсом смотрящим в ISP и туннелем:
```
firewall-cmd --direct --permanent --add-rule ipv4 filter FORWARD 0 -i ens160 -o iptunnel -j ACCEPT
firewall-cmd --direct --permanent --add-rule ipv4 filter FORWARD 0 -i iptunnel -o ens192 -j ACCEPT
```
Открываем порты OSPF на HQ-R и BR-R:
```
firewall-cmd --permanent --zone=trusted --add-port=89/tcp
firewall-cmd --permanent --zone=trusted --add-port=89/udp
```
А так же нужно не забыть добавить туннель в зону trusted на HQ-R и BR-R:
```
firewall-cmd --permanent --zone=trusted --add-interface=tun1
```
>Так же стоит отключить NetworkManager:
>```
>systemctl disable network.service NetworkManager
>```
### Настройка тунелля между HQ-R и BR-R
заходим в файл 
```
vim /etc/net/ifaces/tun1/options
```
если нет то создаем файл
```
mkdir /etc/net/ifaces/tun1
```
и заполняем его 
```
TYPE=iptun
TUNTYPE=gre
TUNLOCAL=192.168.0.161
TUNREMOTE=102.168.0.169
TUNOPTIONS='ttl 64'
HOST=ens?192
```
Потом добавляем ip адрес на туннель
```
echo 4.4.4.1/24 > /etc/net/ifaces/tun1/ipv4address
```
И перезагужаем службу
```
systemctl restart network
```
### Настройка динам. маршрутизации FRR   

устанавливаем пакеты 
```
apt-get update
apt-get -y install frr
```
включаем автозагругку
```
systemctl enable --now frr
```
заходим в даемона
```
vim /etc/frr/daemons
```
> ставим ospfd=yes
заходим
```
vtysh
```
и пишем 
```
config 
route ospf
```
```
passive-interface default
```
``` 
net 192.168.0.128/27 area 0
net 192.168.0.161/30 area  0
```
```
exit
```
```
int tun1 
ip ospf network point-to-point 
no ip ospf passive 
```
сохраняем все
```
do w
```
Иногда настройки vtysh слетают, для этого заходим в:
```
nano /etc/frr/frr.conf
```
И добавляем после ipv6 forwarding такую строчку:
```
ip forwarding
```


### настройка dhcp сервера на HQ-R
### Создание пользователей
Настройте локальные учётные записи на всех устройствах в соответствии с таблицей.

|Учётная запись|	Пароль|	Примечание|
|--------------|--------|-----------|
|Admin|	P@ssw0rd|	CLI, HQ-SRV, HQ-R|
|Branch admin|	P@ssw0rd|	BR-SRV, BR-R|
|Network admin|	P@ssw0rd|	HQ-R, BR-R, HQ-SRV|

Пользователь admin на HQ-SRV
```
adduser admin
```
```
usermod -aG wheel admin
```
```
passwd admin
P@ssw0rd
P@ssw0rd
```
Проверка пользователей с помощью команды
```
vim /etc/passwd
```
### Измерить пропускную способность между двумя узлами HQ-R ISP по средствам утилиты iperf3.
скачиваем пакеты
```
apt-get -y install iperf3
```
ISP как сервер:
>если надо открыть порт
```
>iptables -A INPUT -p tcp --dport 5201 -j ACCEPT
```
```
iperf3 -s
```
HQ-R:
```
iperf3 -c 192.168.0.161 -f M
```
```
[ID] Interval      Transfer   Bitrate        Retr Cwnd
[ 5] 0.00-1.00 sec 345 MBytes 344 MBytes/sec    0 538 KBytes
[ 5] 1.00-2.00 sec 338 MBytes 338 MBytes/sec    0 676 KBytes
[ 5] 3.00-4.00 sec 341 MBytes 341 MBytes/sec    0 749 KBytes
```
### backup скрипты для сохранения конфигурации сетевых устройств HQ-R b BR-R
создать папку для backup-а
```
mkdir /etc/networkbackup
```
Заход в планировщик заданий:
```
EDITOR=nano crontab -e
```
минута | час | день | месяц | день недели | "команда, например reboot":
```
9 15 * * * cp /etc/frr/frr.conf /etc/networkbackup
```
```
ls /etc/networkbackup
```
```
frr.conf
```
### SSH 
