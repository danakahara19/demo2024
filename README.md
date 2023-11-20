# demo2024 Haertdinov Daniel
## Задание demo2024 №1
Выполните базовую настройку всех устройств:
1. Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian
2. Присвоить имена в соответствии с топологией
3. Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.
4. Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.
5. Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.
### Тoпология
![дравио](https://github.com/danakahara19/demo2024/assets/148867574/5884719f-78e0-4729-a06c-44268ae0c34e)
### Таблица ip адресов
|Имя устройства|Интерфейс|IPv4/IPv6    |Маска/Префикс|Шлюз         |
|--------------|---------|-------------|-------------|-------------|
|ISP           |ens192   |192.168.0.170|/30          |             |
|              |ens224   |192.168.0.162|/30          |             |
|              |ens256   |10.12.32.6   |/24          |             |
|HQ-R          |ens224   |192.168.0.5  |/25          |             |
|              |ens192   |192.168.0.169|/30          |192.168.0.170|
|BR-R          |ens192   |192.168.0.136|/27          |             |
|              |ens224   |192.168.0.161|/30          |192.168.0.162|
|HQ-SRV        |ens192   |192.168.0.8  |/25          |             |
|BR-SRV        |ens192   |192.168.0.135|/27          |             |

**Расписал ip адреса с помощью команды**  
```
nano /etc/network/interfaces
```    
**что бы узнать ip  нужна команда**  
```
ip a
```
**После этого перезагрузка системы**
```
systemctl restart networking 
```

#### ISP2 
```
iface ens192 inet static  
address 192.168.0.173  
netmask 255.255.255.252    

auto ens224  
iface ens224 inet static  
address 192.168.0.166  
netmask 255.255.255.252  

auto ens256  
iface ens256 inet static  
address 10.12.32.6  
netmask 255.255.255.255  
gateway 10.10.200.200  
```
#### BR-R
```
auto ens192  
iface ens192 inet static  
address 192.168.0.136  
netmask 255.255.255.224  

auto ens224  
iface ens224 inet static  
address 192.168.0.165  
netmask 255.255.255.252  
gateway 192.168.0.166  
```
#### HQ-R
```
iface ens192 inet static
address 192.168.0.172
netmask 255.255.255.252
gateway 192.168.0.173  

auto ens224  
iface ens224 inet static  
address 192.168.0.5  
netmask 255.255.255.128  
```
**Просмотр статуса сети**
```
systemctl status networking
```
### №1.2 Настройка NAT на ISP, BR-R, HQ-R
Скачиваем 
```
apt install iptables
```
Смотрим
```
nano /etc/sysctl.conf
```
должно показать `net.ipv4.ip_forward=1`    
Пишем
```
sysctl -p
```
```
iptables -A POSTROUTING -t nat -j MASQUERADE
```
```
nano /etc/network/if-pre-up.d/nat
```
Потом прописываем
```
#!/bin/sh
/sbin/iptables -A POSTROUTING -t nat -j MASQUERADE
```
Проверяем
```
chmod +x /etc/network/if-pre-up.d/nat
```
### №1.2 Настройка внутренней динамической маршрутизации по средствам FRR на ISP, BR-R, HQ-R
Сначала надо установить FRR
```
apt-get install update
apt-get install frr
```  
```
nano /etc/frr/daemons
```
`ospfd=no` заменить на 
```
ospfd=yes
```
```
systemctl restart frr
```
заходим в `vtysh`
```
sh int br
```
```
conf t
```
```
router ospf
```
```
net 192.168.0.169 area 0
net 192.168.0.161 area 0
```
```
sh ip ospf neigh
```
### №1.3 Настройка DHCP IP-адресов на роутере HQ-R
Установка DHCP
```
apt install isc-dhcp-server
```        
Вход в конфиг
```
nano /etc/default/isc-dhcp-server
```
Указываю интерфейс
```
INTERFACE="ens192"
```
Вход в конфицурацию 
```
nano /etc/dhcp/dhcpd.conf
```
настраиваем IP
```
subnet 192.168.0.0 netmask 255.255.255.0 {
 range 192.168.0.6 192.168.0.168;
 optiondomain-name-servers 8.8.8.8 8.8.4.4;
 option routers 192.168.0.1;
 option s ubnet-mask 255.255.255.252;
}
```
На HQ-SRV в конфиге поставить DHCP
```
auto ens192
iface ens192 inet dhcp
#address 192.168.0.6
#netmask 255.255.255.128
#gateway 192.168.0.5
```
Перезагружаем dhcp и сервер
```
systemctl restart isc-dhcp-server.service
```
```
systemctl restart networking
```
### №1.4 Настройка локальных учетных записей на всех устройствах
|Учетная запись|Пароль  |Примечание      |
|--------------|--------|----------------|
|Admin         |P@ssw0rd|CLI HQ-SRV HQ-R |
|Branch admin  |P@ssw0rd|BR-SRV BR-R     |
|Network admin |P@ssw0rd|HQ-R BR-R BR-SRV|

Cоздание пользователя 
```
useradd Admin
```
Создание пароля
```
passwd Admin
```
Проверка
```
nano /etc/passwd
```
Так создаем и на остальных устройствах

### №1.5 При помощи Iperp3 измерить пропускную способонсть сети между HQ-SRV b и BR-SRV
Устанавливаем пакеты
```
apt install iperf3
```
```
apt install ufw
```
Задаем порт на HQ-SRV и не выходим пока не подключимся
```
iperf3 -s -p 5201
```
На BR-SRV ставим по умолчанию такой же порт
```
ufw allow 5201
```
Теперь на BR-SRV c помощью этой команды можно подключаться
```
iperf3 -c 192.168.0.8 -p 5201
```
![iperf3](https://github.com/danakahara19/demo2024/assets/148867574/28fbbbca-9652-45d4-8edb-7d277cb01fe3)

### №1.6 Сoставить backup скрипты на HQ-R BR-R
Скачал rsync
```
apt install rsync -y
```
каталог для cохранения backup-ов 
```
mkdir /etc/ffr/backup
```
Зашел в список скриптов 
```
crontab -e
```
поставил в него скрипт для backup-a
```
0 0 * * * rsync -avzh /etc/frr/frr.conf /etc/frr/backup
```
сохраняю и выхожу из списка
>ctrl+s ctrl+z

Проверка backup скрипта
```
cd /etc/frr/backup
```
смотрим какие файлы в каталоге
```
ls -a
```
подверждаем работу скрипта
<img width="322" alt="Снимок экрана 2023-11-20 в 16 30 14" src="https://github.com/danakahara19/demo2024/assets/148867574/df9728d3-67d9-414f-a62c-d060b73f8de1">
