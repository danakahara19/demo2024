
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
firewall-cmd --permanent --zone=public --add-interface=ens33
```
Правила к входяшим пакетам
```
firewall-cmd --permanent --zone=trusted --add-interface=ens34
```
Включаем NAT
```
firewall-cmd --permanent --zone=public --add-masquerade
```
Сохраняем
```
firewall-cmd --reload
```
### Настройка тунелля между HQ-R и BR-R

