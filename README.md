# PiwuKrivo
piwukrivo is a d3m0d3m0d3m0 for s4s4s4

#РЕШЕБНИК:
1 модуль:
ISP (Astra Linux):

hostnamectl set-hostname isp; exec bash

Открыть в mcedit:
mcedit /etc/network/interfaces
Туда писать:
auto eth0 iface eth0 inet dhcp auto eth1 iface eth1 inet static address 172.16.30.1/28 */change/* auto eth2 iface eth2 inet static address 172.16.40.1/28 */change/*

systemctl restart networking

Расскоментировать строку:
mcedit /etc/sysctl.conf 
net.ipv4.ip_forward=1
sysctl -p

iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate INVALID -j DROP
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables-save > /root/iptables_rules
export EDITOR=mcedit
crontab -e
оставить пустую строку после:
@reboot /sbin/iptables-restore < /root/iptables_rules

HQ-RTR (Astra Linux):

hostnamectl set-hostname hq-rtr.au-team.irpo; exec bash

Редактировать файл:
mcedit /etc/network/interfaces
Содержимое :
auto eth0
iface eth0 inet static
address 172.16.30.2/28
gateway 172.16.30.1
auto eth1 iface
eth1 inet manual
auto eth1.112
iface eth1.112
inet static address 192.168.1.1/27
vlan-raw-device
eth1 auto eth1.212 
iface eth1.212 inet static
address 192.168.2.1/28 
vlan-raw-device
eth1 auto eth1.812
iface eth1.812
inet static address 192.168.99.1/29
vlan-raw-device eth1
auto gre1
iface gre1 inet tunnel
address 10.0.0.1
netmask 255.255.255.252
mode gre local 172.16.30.2
endpoint 172.16.40.2
ttl 255

systemctl restart networking

ФРР:
Настраиваем DNS и репозитории для установки FRR:
mcedit /etc/resolv.conf:
nameserver 8.8.8.8 
mcedit /etc/apt/sources.list:
#deb http://dl.astralinux.ru/astra/satble/2.12_x86-64/repository/ orel main contrib non-free deb [trusted=yes] http://archive.debian.org/debian buster main
apt update -y apt install frr -y
ОСПФ:

mcedit /etc/frr/daemons:
ospfd=yes

systemctl restart frr

vtysh
conf t
router ospf 
            network 10.0.0.0/30 area 0
            network 192.168.1.0/27 area 0
            network 192.168.2.0/28 area 0
            network 192.168.99.0/29 area 0
exit
int gre1 
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 123qweR%
exit
do wr mem
quit
 NAT и DHCP :

mcedit /etc/sysctl.conf:
net.ipv4.ip_forward=1

sysctl -p

iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate INVALID -j DROP
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables-save > /root/iptables_rules
export EDITOR=mcedit
crontab -e:
пустая строка после этого:
@reboot /sbin/iptables-restore < /root/iptables_rules


apt update -y && apt install isc-dhcp-server -y
mcedit /etc/dhcp/dhcpd.conf:

subnet 192.168.2.0
netmask 255.255.255.240 { range 192.168.2.2 192.168.2.14;
option domain-search "au-team.irpo";
option routers 192.168.2.1;
option domain-name-servers 192.168.1.2; }

mcedit /etc/default/isc-dhcp-server:
Изменить:
INTERFACESv4="eth1.212"

systemctl restart isc-dhcp-server


useradd net_admin
-m passwd net_admin
P@ssw0rd
P@ssw0rd
mcedit /etc/sudoers:
net_admin ALL=(ALL:ALL) NOPASSWD: ALL

BR-RTR (Astra Linux):

hostnamectl set-hostname br-rtr.au-team.irpo; exec bash
mcedit /etc/network/interfaces:

auto eth0
iface eth0 inet static
address 172.16.40.2/28
gateway 172.16.40.1
auto eth1
iface eth1 inet static
address 192.168.3.1/28
auto gre1
iface gre1 inet tunnel
address 10.0.0.2
netmask 255.255.255.252
mode gre local 172.16.40.2
endpoint 172.16.30.2
ttl 255

systemctl restart networking

OSPF:

mcedit /etc/resolv.conf:
nameserver 8.8.8.8

mcedit /etc/apt/sources.list:
#deb http://dl.astralinux.ru/astra/satble/2.12_x86-64/repository/ orel main contrib non-free
deb [trusted=yes] http://archive.debian.org/debian buster main

apt update -y apt install frr -y

mcedit /etc/frr/daemons
ospfd=yes

systemctl restart frr

vtysh
conf t
router ospf network 10.0.0.0/30 area 0
network 192.168.3.0/28 area 0
exit
int gre1
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 123qweR%
exit
do wr mem
quit

NAТ:
mcedit /etc/sysctl.conf:
net.ipv4.ip_forward=1

sysctl -p
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate INVALID -j DROP
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables-save > /root/iptables_rules
export EDITOR=mcedit
crontab -e
@reboot /sbin/iptables-restore < /root/iptables_rules

useradd net_admin -m
passwd net_admin
P@ssw0rd P@ssw0rd
mcedit /etc/sudoers net_admin:
ALL=(ALL:ALL) NOPASSWD: ALL


mcedit /etc/resolv.conf:
nameserver 192.168.1.2

4. HQ-SRV (Альт Сервер 10)

hostnamectl set-hostname hq-srv.au-team.irpo; exec bash

структура папок:
cd /etc/net/ifaces
cp -r enp0s3/ enp0s3.112


mcedit /etc/net/ifaces/enp0s3.112/options:

BOOTPROTO=static
TYPE=vlan
VID=112
HOST=enp0s3
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=yes
DISABLED=no
CONFIG_WIRELESS=no
CONFIG_IPV4=yes

mcedit /etc/net/ifaces/enp0s3.112/ipv4address:

192.168.1.2/27 

mcedit /etc/net/ifaces/enp0s3.112/ipv4route:

default via 192.168.1.1 

systemctl restart network

export EDITOR=mcedit
crontab -e 
@reboot /bin/systemctl restart network @reboot /bin/systemctl
restart dnsmasq

useradd sshuser -u 2012
passwd sshuser
P@ssw0rd P@ssw0rd 
usermod -aG wheel sshuser
mcedit /etc/sudoers:

WHEEL_USERS ALL=(ALL : ALL) NOPASSWD: ALL


apt-get install openssh-server -y 
systemctl enable --now sshd
mcedit /etc/openssh/sshd_config:

Port 2012 
MaxAuthTries 2
AllowUsers
sshuser
PermitRootLogin no
Banner /root/banner

mcedit /root/banner:

Authorized access only

systemctl restart sshd

apt-get update && apt-get install dnsmasq -y

mcedit /etc/dnsmasq.conf:

interface=enp0s3.112
server=77.88.8.7
domain=au-team.irpo
listen-address=192.168.1.2
no-resolv
no-hosts
address=/hq-rtr.au-team.irpo/192.168.1.1
ptr-record=1.1.168.192.in-addr.arpa,hq-rtr.au-team.irpo
address=/br-rtr.au-team.irpo/192.168.3.1
address=/hq-srv.au-team.irpo/192.168.1.2
ptr-record=2.1.168.192.in-addr.arpa,hq-srv.au-team.irpo
address=/hq-cli.au-team.irpo/192.168.2.2
ptr-record=2.2.168.192.in-addr.arpa,hq-cli.au-team.irpo
address=/br-srv.au-team.irpo/192.168.3.2
ptr-record=2.3.168.192.in-addr.arpa,br-srv.au-team.irpo
address=/docker.au-team.irpo/172.16.30.1
address=/web.au-team.irpo/172.16.40.1


mcedit /etc/net/ifaces/enp0s3.112/resolv.conf:

nameserver 127.0.0.1

systemctl restart dnsmasq

BR-SRV (Альт Сервер 10):

hostnamectl set-hostname br-srv.au-team.irpo; exec bash
mcedit /etc/net/ifaces/enp0s3/options:

BOOTPROTO=static 
TYPE=eth
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=yes
DISABLED=no
CONFIG_WIRELESS=no
CONFIG_IPV4=yes

mcedit /etc/net/ifaces/enp0s3/ipv4address:

192.168.3.2/28

mcedit /etc/net/ifaces/enp0s3/ipv4route:

default via 192.168.3.1

systemctl restart network

export EDITOR=mcedit 
crontab -e
@reboot /bin/systemctl restart network

5.2.
useradd sshuser -u 2012
 passwd sshuser P@ssw0rd P@ssw0rd 
 usermod -aG wheel sshuser 
 mcedit /etc/sudoers:
 WHEEL_USERS ALL=(ALL : ALL) NOPASSWD: ALL

apt-get install openssh-server -y systemctl enable --now sshd
mcedit /etc/openssh/sshd_config:

Port 2012
MaxAuthTries 2
AllowUsers sshuser
PermitRootLogin no
Banner /root/banner

mcedit /root/banner
Authorized access only

systemctl restart sshd

mcedit /etc/resolv.conf 
nameserver 192.168.1.2

  HQ-CLI (Альт Рабочая станция 10)

hostnamectl set-hostname hq-cli.au-team.irpo; exec bash
cd /etc/net/ifaces cp -r enp0s3/ enp0s3.212 
mcedit /etc/net/ifaces/enp0s3.212/options 

BOOTPROTO=dhcp 
TYPE=vlan
VID=212 
HOST=enp0s3
DISABLED=no
CONFIG_IPV4=yes

systemctl restart network

export EDITOR=mcedit
crontab -e
@reboot /bin/systemctl restart network


mcedit /etc/resolv.conf
nameserver 192.168.1.2

2 МОДУЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛЛ

Samba (BR-SRV)

apt-get update apt-get install task-samba-dc -y
rm -rf /etc/samba/smb.conf
samba-tool domain provision

Realm: AU-TEAM.IRPO
Domain: AU-TEAM
Server Role: dc
DNS backend: SAMBA_INTERNAL
DNS forwarder: 192.168.1.2
Пароль администратора: 123qweR%
cp /var/lib/samba/private/krb5.conf
/etc/krb5.conf
systemctl enable --now samba

mcedit /etc/samba/smb.conf
В блок [global] :
interfaces = lo enp0s3
bind interfaces only = yes
Итоговый вид /etc/samba/smb.conf:

[global] 
dns forwarder = 192.168.1.2
netbios name = BR-SRV
realm = AU-TEAM.IRPO
server role = active directory
domain controller workgroup = AU-TEAM
interfaces = lo enp0s3
bind interfaces only = yes
[sysvol] path = /var/lib/samba/sysvol
read only = No
[netlogon] path = /var/lib/samba/sysvol/au-team.irpo/scripts
read only = No

systemctl restart samba samba-tool domain info 127.0.0.1

 Настройка DNS-форвардинга на HQ-SRV:
mcedit /etc/dnsmasq.conf
Добавить строку:
server=/au-team.irpo/192.168.3.2

systemctl restart dnsmasq

Итоговый вид кайфовых строк /etc/dnsmasq.conf

interface=enp0s3.112
server=/au-team.irpo/192.168.3.2
server=77.88.8.7 
domain=au-team.irpo
listen-address=192.168.1.2 
no-resolv
no-hosts

Ввод клиента в домен (HQ-CLI):

su - acc

Действия в acc:

Пользователи - Аутентификация

Выбрать "Домен Active Directory"

Домен: au-team.irpo

Галочка: "Восстановить файлы конфигурации по умолчанию"

Применить, ввести пароль: 123qweR%

 Создание пользователей  (BR-SRV):

samba-tool user create hquser1 123qweR% 
samba-tool user create hquser2 123qweR%
samba-tool user create hquser3 123qweR% 
samba-tool user create hquser4 123qweR%
samba-tool user create hquser5 123qweR% 
samba-tool group add hq 
samba-tool group addmembers hq hquser1,hquser2,hquser3,hquser4,hquser5

Установка схемы LDAP (BR-SRV):

apt-repo add rpm http://altrepo.ru/local-p10 noarch local-p10
apt-get update apt-get install sudo-samba-schema -y sudo-schema-apply
Вводим пароль:
123qweR%
Если ошибка получения Керберос изменить:
/etc/krb5.conf:
includedir /etc/krb5.conf.d/ 
[logging]
default = FILE:/var/log/krb5libs.log
kdc = FILE:/var/log/krb5kdc.log
admin_server = FILE:/var/log/kadmind.log 
[libdefaults] 
dns_lookup_kdc = false 
dns_lookup_realm = false 
ticket_lifetime = 24h
renew_lifetime = 7d
forwardable = true
rdns = false
default_realm = AU-TEAM.IRPO 
default_ccache_name = KEYRING:persistent:%{uid} 
[realms] 
AU-TEAM.IRPO = { kdc = 192.168.3.2 admin_server = 192.168.3.2 default_domain = au-team.irpo } 
[domain_realm]
.au-team.irpo = AU-TEAM.IRPO au-team.irpo = AU-TEAM.IRPO


create-sudo-rule

Заполняем правила:

Имя правила: hq_rules
sudoHost: ALL
sudoCommand: /usr/bin/id
sudoUser: %hq
Сохраняем на ОК
Создание правила sudo через ADMC (HQ-CLI):
apt-get update
apt-get install admc -y 
kinit administrator


123qweR%


admc

В интерфейсе ADMC:
Настройки - Дополнительные возможности
Найти: sudoers - hq-rules
Атрибуты: sudoCommand добавьте через "Изменить"/bin/grep и /bin/cat
Атрибуты: sudoOption добавьте через "Изменить" !authenticate
Сохраняем и закрываем ADMC

 SSSD на клиенте (HQ-CLI):
 
control sudo public
apt-get install libsss_sudo -y

mcedit /etc/sssd/sssd.conf:

[sssd] 
config_file_version = 2 
services = nss, pam, sudo 
user = _sssd 
domains = AU-TEAM.IRPO 
[nss]
[pam] 
[domain/AU-TEAM.IRPO] 
sudo_provider = ad
id_provider = ad 
auth_provider = ad
chpass_provider = ad
access_provider = ad 
default_shell = /bin/bash
fallback_homedir = /home/%d/%u 
debug_level = 0


mcedit /etc/nsswitch.conf
Добавить в конец:
sudors: files sss

systemctl restart sssd
Настройка RAID 5 (HQ-SRV):
Выключаем машину HQ-SRV. Добавляем в носители 3 диска размер по 1ГБ

lsblk mdadm --create /dev/md2 --level=5 --raid-device=3 --spare-devices=0 /dev/sdb /dev/sdc /dev/sdd 
mdadm --detail --scan > /etc/mdadm.conf cat /etc/mdadm.conf

fdisk /dev/md2 
В fdisk: n -> p -> Enter -> Enter -> Enter -> w
mkfs.ext4 /dev/md2p1 
mkdir /raid mount /dev/md2p1 /raid */change/* lsblk

lsblk /dev/md2 



systemctl --force --full edit raid.mount

Итоговый вид файла юнита raid.mount


[Unit] 
Description=Mount RAID5 
[Mount]
What=/dev/md2p1 
Where=/raid 
Type=ext4
Options=defaults
[Install]
WantedBy=multi-user.target

Сохранить и выйти: Shift + ; и написать wq



systemctl enable --now raid.mount
mount -a df -h

NFS (HQ-SRV и HQ-CLI)


Настройка сервера (HQ-SRV)

apt-get install nfs-server -y 
mkdir /raid/nfs 
chmod 777 /raid/nfs
mcedit /etc/exports
Строка экспорта:
/raid/nfs 192.168.2.0/28(rw,no_subtree_check)
Сохраняем изменения и выходим

exportfs -a exportfs -v systemctl enable --now nfs-server systemctl status nfs-server


Настройка клиента (HQ-CLI)

apt-get install nfs-clients -y 
mkdir /mnt/nfs 
systemctl --force --full 
edit mnt-nfs.mount



Итоговый вид файла юнита mnt-nfs.mount


[Unit]
Description=Mount NFS
[Mount] 
What=192.168.1.2:/raid/nfs
Where=/mnt/nfs
Type=nfs 
Options=_netdev,auto 
[Install]
WantedBy=multi-user.target

Сохраняем файл nano сочетанием Ctrl + O и Ctrl + X



systemctl enable --now mnt-nfs.mount 
mount -a df -h
touch /mnt/nfs/vftest

Проверим созданный файл на HQ-SRV:

ls -lah /raid/nfs/


(Chrony)



NTP-сервер на ISP

apt update apt install chrony -y
mcedit /etc/chrony/chrony.conf

Итоговый вид /etc/chrony/chrony.conf (ISP) - Stratum 7 


local stratum 7 
allow 172.16.30.0/28 
allow 172.16.40.0/28 
allow 192.168.1.0/27
allow 192.168.2.0/28 
allow 192.168.3.0/28 

Закоментировать в этом файле строки pool.. и rtcsync. Сохраняем файл



systemctl restart chronyd
systemctl status chronyd 
timedatectl set-ntp 0 
timedatectl


Роутеры (HQ-RTR, BR-RTR):

mcedit /etc/systemd/timesyncd

Итоговый вид /etc/systemd/timesyncd (HQ-RTR)



[Time] NTP=172.16.30.1 

Итоговый вид /etc/systemd/timesyncd (BR-RTR)


[Time] NTP=172.16.40.1 

Серверы (HQ-SRV, BR-SRV) и НQ-CLI:

На серверах уже должна быть включена служба:

systemctl status chronyd

mcedit /etc/chrony.conf

Итоговый вид /etc/chrony.conf (HQ-SRV и НQ-CLI)

server 172.16.30.1 iburst 

Закоментировать в этом файле строки pool.. и rtcsync. Сохраняем файл

Итоговый вид /etc/chrony.conf (BR-SRV)


server 172.16.40.1 iburst 

Закоментировать в этом файле строки pool.. и rtcsync. Сохраняем файл


systemctl restart chronyd chronyc sources

Если "^?", то связи с сервером нету


Ansible на BR-SRV


Подготовка SSH на хостах

Роутеры:

apt install openssh-server -y 
mcedit /etc/ssh/sshd_config

Итоговый вид sshd_config 



Port 2012 
MaxAuthTries 2
PermitRootLogin no
AllowUsers net_admin

Сохраняем изменения и выходим.

systemctl restart sshd

Клиент (HQ-CLI):

useradd sshuser -u 2012 
passwd sshuser 
usermod -aG wheel
sshuser mcedit /etc/sudoers

Раскомментировать: WHEEL_USERS ALL=(ALL) NOPASSWD: ALL

apt-get install openssh-server -y
mcedit /etc/openssh/sshd_config

Итоговый вид sshd_config (HQ-CLI)



Port 2012
MaxAuthTries 2
PermitRootLogin no
AllowUsers sshuser

Сохраняем изменения и выходим.

systemctl restart sshd

Настройка Ansible (BR-SRV)

apt-get install ansible -y 
mcedit /etc/ansible/hosts

Итоговый вид /etc/ansible/hosts (Port 2012)

Читайте внимательно:

[clients]
hq-rtr ansible_host=192.168.1.1 ansible_user=net_admin ansible_port=2012 
br-rtr ansible_host=192.168.3.1 ansible_user=net_admin ansible_port=2012 
hq-srv ansible_host=192.168.1.2 ansible_user=sshuser ansible_port=2012 
hq-cli ansible_host=192.168.2.2 ansible_user=sshuser ansible_port=2012 


13.3. Ключи и проверка (пароль - P@ssw0rd)
ssh-keygen
ssh-copy-id net_admin@192.168.1.1 -p 2012 
ssh-copy-id net_admin@192.168.3.1 -p 2012 
ssh-copy-id sshuser@192.168.1.2 -p 2012 
ssh-copy-id sshuser@192.168.2.2 -p 2012 



mcedit /etc/ansible/ansible.cfg

В блок [defaults]:

interpreter_python = python3



ansible all -m ping



Docker (BR-SRV)




apt-get install docker-engine docker-compose -y
systemctl enable --now docke
r systemctl status docker


Импорт образов
НЕОБХОДИМО сначала выключить машину и добавить Additional.iso образ диска

mkdir /mnt/add_cd
mount /dev/sr0 /mnt/add_cd 
cp -r /mnt/add_cd/docker ~/docker/
docker image load -i /root/docker/site_latest.tar
docker image load -i /root/docker/mariadb_latest.tar
docker images

Если ошибка на image:

cd ~/docker/ tar -tvf site_latest.tar # Если выдаст ошибку вроде 'unexpected EOF' или 'gzip: stdin: unexpected end of file', файл поврежден. rm site_latest.tar #если повержден cp /mnt/add_cd/docker/site_latest.tar ~/docker/ tar -tvf site_latest.tar docker image load -i site_latest.tar



mkdir testapp cd testapp/

mcedit docker-compose.yaml

Итоговый вид docker-compose.yaml (DB: testdb2, User: test2c, Port: 8082) 


<img width="507" height="761" alt="image" src="https://github.com/user-attachments/assets/7454ccac-db88-4138-acb6-a44935d6a09e" />
<img width="494" height="315" alt="image" src="https://github.com/user-attachments/assets/68514a59-dea0-4d98-af34-c9392e52118f" />


services:
testapp: 
image:
site:latest
container_name: testapp 
restart: always 
depends_on: - 
db ports: - 8082:8000 
environment: 
DB_TYPE:
maria DB_HOST: db DB_NAME: testdb2 */change/* DB_PORT: 3306 DB_USER: test2c 
DB_PASS: P@ssw0rd db: image: mariadb:10.11 container_name: db restart: always environment: MARIADB_DATABASE: testdb2 
MARIADB_USER: test2c 
MARIADB_PASSWORD: P@ssw0rd MARIADB_ROOT_PASSWORD: 123qweR% volumes: - /root/testapp/db_data:/var/lib/mysql volumes: db_data:


docker compose up -d docker ps



Зайти через клиент по адресу в браузере, для теста создать несколько записей на сайте
Перезайти на сайт и проверить, что они сохраняются
http://192.168.3.2:8082 


LAMP (HQ-SRV)


apt-get install apache2 -y
apt-get install mariadb -y
apt-get install php8.2 apache2-mod_php8.2 php8.2-mysqli -y
systemctl enable --now httpd2 systemctl
enable --now mariadb


НЕОБХОДИМО сначала выключить машину и добавить Additional.iso образ диска

mkdir /mnt/add_cd mount /dev/sr0 /mnt/add_cd
cp -r /mnt/add_cd/web /root/web


mysql_secure_installation
Нажимаем по порядку: Enter --> y --> y --> 123qweR% --> 123qweR% --> y(до конца)
mysql
SQL запросы:
CREATE DATABASE webdb;
exit
mysql webdb < /root/web/dump.sql mysql CREATE USER 'web2c'@'%' IDENTIFIED BY 'P@ssw0rd'; 
GRANT ALL PRIVILEGES ON webdb.* TO 'web2c'@'%'; 
FLUSH PRIVILEGES;
exit



mcedit /root/web/index.php

Изменить переменные:

$servername = "localhost"
; $username = "web2c"; 
$password = "P@ssw0rd";
$dbname = "webdb";


cp /root/web/index.php /var/www/html/
rm -f /var/www/html/index.html
cp /root/web/logo.png /var/www/html/logo.png a2enmod mod_php8.2 
systemctl restart httpd2


Зайти через клиент по адресу в браузере, для теста создать несколько записей на сайте
Перезайти на сайт и проверить, что они сохраняются
http://192.168.1.2


NAT и Nginx (ISP)



HQ-RTR:

iptables -t nat -A PREROUTING -d 172.16.30.2 -p tcp --dport 8082 -j DNAT --to-destination 192.168.1.2:80
iptables -t nat -A PREROUTING -d 172.16.30.2 -p tcp --dport 2012  -j DNAT --to-destination 192.168.1.2:2012
 iptables -t nat -L iptables-save > /root/iptables_rules
 
BR-RTR:

iptables -t nat -A PREROUTING -d 172.16.40.2 -p tcp --dport 8082 -j DNAT --to-destination 192.168.3.2:8082 
iptables -t nat -A PREROUTING -d 172.16.40.2 -p tcp --dport 2012 
-j DNAT --to-destination 192.168.3.2:2012 
iptables -t nat -L iptables-save > /root/iptables_rules

Nginx Reverse Proxy (ISP)

apt install nginx -y
mcedit /etc/nginx/sites-available/nginx-proxy

Итоговый вид /etc/nginx/sites-available/nginx-proxy


Я НЕ БУДУ СТАВИТЬ ТАБУЛЯЦИЮ, С БОГОМ НАХУЙ

server {
            listen 80;
            server_name web.au-team.irpo;
            location / {
                        proxy_pass http://172.16.30.2:8082;
                        proxy_set_header Host $host;
                        proxy_set_header X-Real-IP $remote_addr;
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_set_header X-Forwarded-Proto $scheme;
   }
} server {
            listen 80;
            server_name docker.au-team.irpo;
            location / {
                        proxy_pass http://172.16.40.2:8082;
                        proxy_set_header Host $host;
                        proxy_set_header X-Real-IP $remote_addr;
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_set_header X-Forwarded-Proto $scheme;
    }
}


nginx -t

ln -s /etc/nginx/sites-available/nginx-proxy /etc/nginx/sites-enabled/nginx-proxy

systemctl restart nginx

Зайти через клиент по адресу в браузере
http://web.au-team.irpo
http://docker.au-team.irpo

 Веб-аутентификация (User Kozmac) 
 
apt install apache2-utils -y 
htpasswd -c /etc/nginx/.htpasswd Kozmac 
mcedit /etc/nginx/sites-available/nginx-proxy

В блок server_name web.au-team.irpo добавьте:

auth_basic "Web-authorization";
auth_basic_user_file /etc/nginx/.htpasswd;

nginx -t 
systemctl restart nginx



Зайти через клиент по адресу в браузере.Вводим имя Kozmac и пароль P@ssw0rd 

http://web.au-team.irpo


Установка Яндекс Браузера (HQ-CLI)

su - apt-get install yandex-browser-stable -y

Запуск: Меню -> Интернет -> Yandex Browser


Модуль 3: 
Импорт пользователей (BR-SRV)


Выполнить монтирование Additional.iso в директорию /mnt:
mount /dev/sr0 /mnt/
mcedit /root/import.sh
Создание скрипта импорта:


#!/bin/bash
csv_file="$1"
# Create OU
awk -F ';' 'NR>1 {print $5}' "$csv_file" | sort | uniq | while read ou; do
            samba-tool ou add OU="$ou",DC=au-team,DC=irpo;
done
# Create Users
while IFS=";" read -r firstName lastName role phone ou street zip city country password; do
            if [ "$firstName" == "First Name" ]; then
                        continue 
            fi
            username="$(echo $firstName | cut -c1).$(echo $lastName)"
            samba-tool user add "$username" Password1 \
                        --given-name="$firstName" \
                        --surname="$lastName" \
                        --telephone-number="$phone" \
                        --job-title="$role" \
                        --userou="OU=$ou"
            samba-tool user setexpiry "$username" --noexpiry
done < "$csv_file"

chmod +x /root/import.sh /root/import.sh /mnt/Users.csv




Создание домашних директорий (HQ-CLI)

mcedit /root/im.sh
Я НЕ БУДУ ТАБУЛЯЦИЮ ТУТ ДЕЛАТЬ НЕТ

exclude_users="administrator guest krbtgt hquser1 hquser2 hquser3 hquser4 hquser5"

wbinfo -u 2>&1 | while read user; do
            echo ">> $user"
            if [[ "$user" == *\$* ]] || [[ -z "$user" ]]; then
                        continue
            fi
            if echo "$exclude_users" | grep -qw "$user"; then
                        echo "Пропускаем: $user (исключён)"
                        continue
            fi
            sudo mkdir -p "/home/AU-TEAM.IRPO/$user"
            sudo cp -r "/etc/skel/." "/home/AU-TEAM.IRPO/$user"
            sudo chown "$user" "/home/AU-TEAM.IRPO/$user" 
            sudo chmod 700 "/home/AU-TEAM.IRPO/$user"
            echo "Создано: $user"
done

chmod +x /root/im.sh /root/im.sh

Проверить: su - любой пользователь, и hquser тоже должен входить


Настройка Шифрования туннеля (HQ-RTR)



apt-get update 
apt-get install -y strongswan strongswan-pki libstrongswan-extra-plugins



mcedit /etc/ipsec.conf
табуляция хуйня!!!!!!!!!!!!!
добавить в config setup
charondebug="ike 2, knl 2, cfg 2"
conn %default 
ikelifetime=86400s
keylife=3600s
rekeymargin=3m
keyingtries=%forever
conn hq-to-br
type=transport
keyexchange=ikev2
authby=secret
left=172.16.30.2 
leftid=172.16.30.2 
right=172.16.40.2 
rightid=172.16.40.2 
ike=aes256-sha256-modp2048
esp=aes256-sha256
auto=start
mobike=no 
fragmentation=yes



mcedit /etc/ipsec.secrets

172.16.30.2 172.16.40.2 : PSK "P@ssw0rd" */change/*

сохраняем и выходим

chmod 600 /etc/ipsec.secrets




systemctl restart strongswan 
systemctl enable strongswan


Настройка Шифрования туннеля (BR-RTR)


apt-get update apt-get install -y 
strongswan strongswan-pki
libstrongswan-extra-plugins



mcedit /etc/ipsec.conf

добавить в config setup

charondebug="ike 2, knl 2, cfg 2" conn %default
ikelifetime=86400s 
keylife=3600s
rekeymargin=3m
keyingtries=%forever
conn br-to-hq
type=transport
keyexchange=ikev2 
authby=secret 
left=172.16.40.2 
leftid=172.16.40.2
right=172.16.30.2  rightid=172.16.30.2 
ike=aes256-sha256-modp2048
esp=aes256-sha256
auto=start
mobike=no
fragmentation=yes


mcedit /etc/ipsec.secrets

172.16.40.2 172.16.30.2 : PSK "P@ssw0rd" */change/*


chmod 600 /etc/ipsec.secrets



systemctl restart strongswan
systemctl enable strongswan

Тестирование туннеля (ISP)




apt install tcpdump -y
tcpdump -i eth1 -n esp or udp port 500 or udp port 4500


Настройка принтера (HQ-SRV)


apt-get update && apt-get install -y
cups cups-pdf 
systemctl enable --now cups cupsctl --share-printers --remote-any

Добавление виртуального принтера (HQ-CLI)

sudo lpadmin -p Virtual_PDF -E -v ipp://192.168.1.2:631/printers/cups-pdf -m everywhere
sudo lpoptions -d Virtual_PDF lpstat -d


Через принтеры отправить пробную печать и на HQ-SRV проверить командой:
ls -l /var/spool/cups/

Установка Fail2ban (HQ-SRV)


Установите сам fail2ban и пакет для корректной работы с журналом systemd.
sudo apt-get update && sudo apt-get install fail2ban python3-module-systemd -y 
mkdir /opt/fail2ban

Настройка jail (Ban 3 min) 


Создайте пользовательский файл /etc/fail2ban/jail.d/ssh.conf (он не будет перезаписан при обновлениях) со следующими параметрами:
mcedit /etc/fail2ban/jail.d/ssh.conf

Содержимое файла:

[sshd] enabled = true # Включаем защиту
port = 2012 #Указываем порт SSH (ssh = 2012)
filter = sshd # Используем штатный фильтр для сообщений 
sshd logpath = /opt/fail2ban # Путь к логам в ОС Альт
maxretry = 3 # Количество неудачных попыток до бана (3) 
bantime = 180 # Время бана в секундах (180 = 3 минуты) 
findtime = 600 # Интервал в секундах для подсчета попыток 
backend = systemd # Указываем systemd для чтения логов (важно для ОС Альт)


После создания конфигурации нужно перезапустить службу и проверить её статус:

sudo systemctl restart fail2ban
sudo fail2ban-client status sshd



Настройка сервера логов (HQ-SRV)

На HQ-SRV разрешаем прием логов по UDP и TCP:

apt update && apt install rsyslog -y 
mcedit /etc/rsyslog.d/HQ-SRV.conf

Содержимое файла:

module(load="imudp") 
input(type="imudp" port="514") 
module(load="imtcp") 
input(type="imtcp" port="514")
template(name="RemoteLog" type="string" string="/opt/%HOSTNAME%/%PROGRAMNAME%.log")
*.*;auth,authpriv.none ?RemoteLog

Перезапускаем rsyslog:

systemctl restart rsyslog
systemctl enable --now rsyslog


Настройка клиентов (HQ-RTR, BR-RTR, BR-SRV)

На каждом клиенте создайте файл конфигурации (замените IP_HOSTA_HQ_SRV на IP HQ-SRV):
mcedit /etc/rsyslog.d/hq-rtr.conf

Содержимое файла:
*.warning @IP_HOSTA_HQ_SRV:514 *.warning @@IP_HOSTA_HQ_SRV:514
Перезапустите службу:
systemctl restart rsyslog


Ротация логов на HQ-SRV (Min Size 12MB) 

mcedit /etc/logrotate.d/rsyslog-remote

Содержимое файла:

/opt/*/*.log /opt/*.log { weekly rotate 4 compress delaycompress missingok notifempty minsize 12M */change/* create 0644 root root sharedscripts postrotate /usr/bin/systemctl kill -s HUP rsyslog >/dev/null 2>&1 || true endscript }

Проверьте конфигурацию:

logrotate -d /etc/logrotate.d/rsyslog-remote


На клиенте (например, BR-SRV) выполните:

logger -p user.warning "TEST PIA"

На HQ-SRV проверьте наличие файлов (проверьте папку с именем хоста клиента в /opt), таб 2 раза:
cat /opt/

Должен быть виден лог с TEST PIA.
