## Задание 1
На стенде утановить новую машину. Название Ansible.  
![фтышиду](https://github.com/danakahara19/demo2024/assets/148867574/7e4983c0-c569-4966-9a64-5e1a29c82207)
На витуальной машине в домашнем каталоге создать папку Ansible. В созданной папке создать виртуальное окружение Python3.12.1. Виртуальное окружение поместить в папку **.env**  
![Снимок экрана (5)](https://github.com/danakahara19/demo2024/assets/148867574/a90c628c-7e09-48de-ad0d-3102f66f6abf)
Настроить SSH для удаленного подключения к виртуальной машине  
обновляем пакеты
```
apt update
```
Скачиваем ssh
```
apt install openssh-server
```
Смотрим статус ssh
```
systemctl status ssh
```
сли выкл то включаем его командой
```
systemctl start ssh
```
выключить его
```
systemctl stop ssh
```
Что бы не запускать его каждый раз введите команду 
```
systemctl enable ssh
```
Что бы выключить
```
systemctl disable ssh
```
Посмотреть имя пользователя
```
hostname -I
```
Посмотреть IP-адрес
```
whoami
```
Что бы удаленно подключиться нужна команда
```
ssh ansible@10.12.32.21
```
если не работает попробуйте перезагрузить ssh
```
systemctl restart ssh
```
Зайдите в nano
```
nano /etc/ssh/sshd_config
```
измените строку #permitrootlogin уберите коммент и напишите **yes**
![image](https://github.com/danakahara19/demo2024/assets/148867574/3aa8e504-a0f7-4d8e-95fc-029b137a9b01)
