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
|HQ-SRV        |ens192   |192.168.0.6  |/25          |192.168.0.5  |
|BR-SRV        |ens192   |192.168.0.135|/27          |192.168.0.136|

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
### Настройка NAT на ISP, BR-R, HQ-R
```
apt install iptables
```
```
nano /etc/sysctl.conf
```
должно показать `net.ipv4.ip_forward=1`
```
sysctl -p
```
```
iptables -A POSTROUTING -t nat -j MASQUERADE
```
```
nano /etc/network/if-pre-up.d/nat
```
```
#!/bin/sh
/sbin/iptables -A POSTROUTING -t nat -j MASQUERADE
```
```
chmod +x /etc/network/if-pre-up.d/nat
```
### Настройка внутренней динамической маршрутизации по средствам FRR
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
