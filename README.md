# Welcome to GitHub!

GitHUB заметки. 

Настройки на Alt, если надо сделать интерфейс с Вланом:

mkdir /etc/net/ifaces/enp6s18.330
vim /etc/net/ifaces/enp6s18.330/options

TYPE=vlan
HOST=enp6s18
VID=330
BOOTPROTO=static

echo 192.168.11.82/29 > /etc/net/ifaces/enp6s18.330/ipv4address 
echo default via 192.168.11.81 > /etc/net/ifaces/enp6s18.330/ipv4route
echo nameserver 77.88.8.8 > /etc/net/ifaces/enp6s18.330/resolv.conf

systemctl restart network

OVS заметки
systemctl enable --now openvswitch
systemctl status openvswitch
ovs-vsctl show

ovs-vsctl add-br <ИМЯ SW> - Добавление виртуального коммутатора. Автоматически создается интерфейс с аналогичным именем
ovs-vsctl add-port <ИМЯ SW> <ИМЯ ПОРТА> - Добавление порта в коммутатор (порт должен быть включен) закидываем все физ. порты и логические
ovs-vsctl add-port <ИМЯ SW> <ИМЯ ВИРТУАЛЬНОГО ПОРТА> - Добавление виртуального порта в коммутатор (выйдет ошибка, что нет такого порта)

ovs-vsctl set Interface <ИМЯ ВИРТУАЛЬНОГО ПОРТА> type=internal - Устанавливаем виртуальному порту тип internal
ovs-vsctl set Port <ИМЯ ПОРТА> tag=<НОМЕР VLAN>
ovs-vsctl set Port <ИМЯ ПОРТА> trunks=<VLAN1>,<VLAN2>

ovs-appctl stp/show
ovs-vsctl set Bridge <ИМЯ SW> stp_enable=yes
ovs-vsctl set Bridge <ИМЯ SW> other_config:stp-priority=<ПРИОРИТЕТ>

На РедОС, что на вирт. порту применился IP, то закидываем:
ip link set name_ports* up
ip address add <IP АДРЕС ВИРТУАЛЬНОГО ПОРТА> dev <ИМЯ ВИРТУАЛЬНОГО ПОРТА>
ip route add default via <АДРЕС ШЛЮЗА>

Можно закинуть в rc.local 

vim /etc/rc.d/rc.local

#!/bin/sh

ip link set name_ports* up
ip address add <IP АДРЕС ВИРТУАЛЬНОГО ПОРТА> dev <ИМЯ ВИРТУАЛЬНОГО ПОРТА>
ip route add default via <АДРЕС ШЛЮЗА>

exit 0

chmod +x /etc/rc.d/rc.local

systemctl enable --now rc-local

Для стп сделать  перезапуск

vim /etc/rc.d/rc.local

#!/bin/sh

systemctl restart openvswitch

exit 0

chmod +x /etc/rc.d/rc.local

systemctl enable --now rc-local

r-br (EcoRouter)


Если Alt или другой, без NM, то настройки через конфиг:

vim /etc/net/ifaces/SW1-HQ/options

TYPE=ovsbr
HOST='enp6s18 enp6s19 enp6s20'
OVS_OPTIONS=’stp_enable=yes other_config:stp-priority=4096’
OVS_EXTRA=’set port enp6s18 trunk=110,220,330 -- set port enp6s19 trunk=110,220,330 -- set port enp6s20 trunk=110,220,330’
 
vim /etc/net/ifaces/mgmt/options
 
TYPE=ovsport
BOOTPROTO=static
CONFIG_IPV4=yes
BRIDGE=SW1-HQ
VID=330
 
Поправить /etc/net/ifaces/default/options 
OVS_REMOVE=no

диски в ЛВМ и Рейд
lsblk - посмотретьб диски
df -h - что использукется 
ОЗДАНИЕ RAID МАССИВА

Для создания RAID массива выполните следующие шаги:

Подготовьте диски для использования в RAID массиве. Это может потребовать форматирования или разделения дисков. Создайте RAID массив с помощью команды, аналогичной следующей:

sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1 (смотреть свои)

Эта команда создает RAID 1 массив с двумя дисками. После этого RAID массив будет создан и начнет синхронизацию.
Управление RAID массивом

Mdadm предоставляет множество команд для управления RAID массивами. Вот некоторые из них:

Просмотр информации о массиве:

sudo mdadm --detail /dev/md0

Добавление нового диска в массив:

sudo mdadm --add /dev/md0 /dev/sdc1

Удаление диска из массива:

sudo mdadm --remove /dev/md0 /dev/sda1


Для различения файловых систем используется указание типа файловой системы после параметра -t или в качестве компонента имени утилиты, например:

Создание фс
mkfs -t xfs /dev/md0

mkdir /opt/ansible 
chmod 777 /opt/ansible

Узнаём универсальный идентификатор диска UUID (Universally Unique Identifier): 
blkid

идент в /etc/fstab 

далее прописать по аналогии как defaults

https://www.altlinux.org/%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0_Fstab


lvm

создать физический том с помощью команды:
sudo pvcreate /dev/vdb
sudo pvcreate /dev/vdc
 создать группу томов
vgcreate {vgname} {pvname}
vgcreate vg /dev/vdb /dev/vdc
Создание логического тома делается такой командой:

sudo lvcreate --size {size} --name {lv-name} {vg-name}
sudo lvcreate --size 5G --name lv-example vg-example
      
Чтобы создать файловую систему xfs, наберите команду:

sudo mkfs.xfs /dev/vg-example/lv-example

Если же вы хотите, чтобы логический том использовал всё свободное место в группе томов, то наберите команду:

sudo lvcreate --extents 100%FREE --name lv-example vg-example

Например, вы хотите смонтировать созданный логический том в папку /opt. В таком случае, добавьте такую строку в файл /etc/fstab:

/dev/vg-example/lv-example  /opt xfs defaults 0 1


Samba и прочее на Alt:

apt-get install bind
apt-get install bind-utils
apt-get install task-samba-dc

при установке системы, задавать полное имя для DC (FQDN)!
 
#Меняем имя на короткое
hostnamectl set-hostname srv1-hq

#Отключаем chroot:
control bind-chroot disabled
 
#Отключаем KRB5RCACHETYPE
vim /etc/sysconfig/bind

KRB5RCACHETYPE="none"
Если нет, то дописать. 

Для Бинда дописать 
в vim /etc/bind/named

в конце
#include "/var/lib/samba/bind-dns/named.conf";

Редактируем бинд
vim /etc/bind/options.conf
recursing-file "/var/run/recursing";

tkey-gssapi-keytab "/var/lib/samba/bind-dns/dns.keytab";
minimal-responses yes;

****
listen-on { any; };
listen-on-v6 { none; };

***
forward first;
forwarders { 77.88.8.8; };


*****
allow-query { any; };

В конце 
category lame-servers { null; };

#Очищаем базы и конфигурацию Samba
 
rm -f /etc/samba/smb.conf
rm -rf /var/lib/samba
rm -rf /var/cache/samba

mkdir -p /var/lib/samba/sysvol


#Пример команды создания контроллера домена au.team в пакетном режиме:
 
samba-tool domain provision --realm=au.team --domain=au --adminpass='P@ssw0rd' --dns-backend=BIND9_DLZ --server-role=dc --use-rfc2307
 
где:
--realm=au.team - имя области Kerberos (LDAP), и DNS имя домена;
--domain=au - имя домена (имя рабочей группы);
--adminpass='P@ssw0rd' - пароль основного администратора домена;
--dns-backen=BIND9_DLZ -  бэкенд DNS-сервера;
--server-role=dc - тип серверной роли;
--use-rfc2307 - позволяет поддерживать расширенные атрибуты типа UID и GID в схеме LDAP и ACL на файловой системе Linux.

systemctl enable --now samba
systemctl enable --now bind


#Заменяем файл krb5.conf, находящийся в каталоге /etc/ на файл, созданный в момент создания домена Samba
 
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
 
#Меняем адрес DNS сервера на 127.0.0.1
 
echo nameserver 127.0.0.1 > /etc/net/ifaces/enp6s18/resolv.conf
 
#Перезагружаем сервер
 
reboot

samba-tool domain info 127.0.0.1

Просмотр предоставляемых служб
 
smbclient -L localhost -Uadministrator

#Проверка имён хостов
 
host -t SRV _kerberos._udp.au.team.
host -t SRV _ldap._tcp.au.team.
host -t A srv1-hq.au.team


#Проверка Kerberos (имя домена должно быть в верхнем регистре):
 
kinit administrator@AU.TEAM
klist


Скрипт
на пользователей

#!/bin/bash

# Создание пользователей и добавление в группы
for ((i=1; i<=30; i++))
do
    username="user$i"
    password="P@ssw0rd"

    if [ $i -le 10 ]; then
        samba-tool user create $username $password
        samba-tool group addmembers group1 $username
        echo "Пользователь $username создан и добавлен в группу group1"
    elif [ $i -le 20 ]; then
        samba-tool user create $username $password
        samba-tool group addmembers group2 $username
        echo "Пользователь $username создан и добавлен в группу group2"
    else
        samba-tool user create $username $password
        samba-tool group addmembers group3 $username
        echo "Пользователь $username создан и добавлен в группу group3"
    fi
done


