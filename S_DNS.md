#Вывести список зон DNS

samba-tool dns zonelist <СЕРВЕР>

Пример

samba-tool dns zonelist 192.168.11.2 -Uadministrator

#Создать зону DNS

samba-tool dns zonecreate <СЕРВЕР> <ЗОНА>

Пример

samba-tool dns zonecreate 192.168.11.2 11.168.192.in-addr.arpa -Uadministrator

Создание обратных зон для каждой сетиЮ пример:

1. Для офиса HQ выделена сеть 192.168.11.0/24 (Пример выше)
2. Для офиса BR выделена сеть 192.168.22.0/24
3. Для офиса DT выделена сеть 192.168.33.0/24

#Добавить новую запись

samba-tool dns add <СЕРВЕР> <ЗОНА> <ИМЯ> <A|AAAA|PTR|CNAME|NS|MX|SRV|TXT> <ДАННЫЕ>

#Пример добавить запись типа A

samba-tool dns add 192.168.11.2 au.team sw1-hq A 192.168.11.82 –Uadministrator

#Пример добавить запись типа PTR для обратной зоны 192.168.11.0/24

samba-tool dns add 192.168.11.2 11.168.192.in-addr.arpa 82 PTR sw1-hq.au.team -Uadministrator
82 - последжний октет IP сервера
sw1-hq.au.team - хостнэйм сервера. 

Повторять сколько потребуется

#Вывести информацию о DNS-записях

samba-tool dns query <сервер> <зона> <имя> <A|AAAA|PTR|CNAME|NS|MX|SOA|SRV|TXT|ALL>

#Пример вывести всю информацию о DNS-записях зоны au.team

samba-tool dns query 192.168.11.2 au.team @ ALL -Uadministrator

#Пример вывести всю информацию о DNS-записях зоны 11.168.192.in-addr.arpa

samba-tool dns query 192.168.11.2 11.168.192.in-addr.arpa @ ALL -U administrator

https://docs.altlinux.org/ru-RU/domain/10.2/html-single/samba/index.html#dns-management

Ввод клиента 

#Обновляем репозитории
apt-get update

#Устанавливаем пакет task-auth-ad-sssd:
apt-get install -y task-auth-ad-sssd
Меню → Центр управления → Центр управления системой) необходимо выбрать пункт Пользователи → Аутентификация.

RED OS 
Системные Ввод ПК в домен
Домен вин /самба
ребут

apt-get install admc

SamBA REDOS
#На время настройки переведите SELinux в режим уведомлений
 setenforce 0
 
#Устанавливаем необходимые пакеты
 dnf install samba*
 dnf install krb5*
 dnf install bind

#Проверяем права на доступ к файлу /etc/krb5.conf (Пользователем-владельцем файла должен быть root, а группой-владельцем – named)
 ls -l /etc/krb5.conf

#При необходимости меняем владельца
 chown root:named /etc/krb5.conf

#Редактируем файл /etc/krb5.conf
 vi /etc/krb5.conf

Поменять дефолт реалм на совй
default_realm = AU.domen

Внести инфо про реалмс,

[realms]
AU.DOMEN = {
		kdc = srv1-hq.au.team
		admin_server = srv1-hq.au.team
}

[domain_realm]
 .au.team = AU.team
 au.team = AU.TEAM 
 
 #Редактируем файл /etc/krb5.conf.d/crypto-policies
 vi /etc/krb5.conf.d/crypto-policies

https://redos.red-soft.ru/base/server-configuring/domain-config/samba-dns-backend-bind9-dlz/admin-samba-bind/


Samba AD на RED OS

https://redos.red-soft.ru/base/server-configuring/domain-config/samba-dns-backend-bind9-dlz/admin-samba-bind/


#Редактируем конфигурационный файл bind
vi /etc/named.conf

options 

... port 53 { any; };
... v6 port 53 { none; };

*****/

forward first;
forwarders { 77.88.8.8 ; };

****
#include "/var/lib/samba/bind-dns/named.conf";

 #Очищаем базы и конфигурацию Samba
 
rm -rf /etc/samba/smb.conf


#Присоединение сервера в качестве вторичного контроллера домена

samba-tool domain join au.team DC -U Administrator --dns-backend=BIND9_DLZ --realm=au.team

https://redos.red-soft.ru/base/server-configuring/domain-config/samba-dns-backend-bind9-dlz/admin-samba-bind/


#Редактируем файл /etc/samba/smb.conf.
vi /etc/samba/smb.conf

idmap_ldb:use rfc2307 = yes
vfs objects = acl_xattr
map acl inherit = yes 
store dos attributes = yes 
nsupdate command = /usr/bin/nsupdate -y
dsbd:schema update allowed = true 


#После внесения изменений в файл /etc/samba/smb.conf выполняем проверку
 testparm

#Включаем и добавляем в автозагрузку службы samba и bind (named)
systemctl enable --now samba
systemctl enable --now named

#Проверьте работу динамического обновления DNS
samba_dnsupdate --verbose --all-names

Создаём директорию

mkdir /opt/data

Задаём права на созданную директорию

chmod 777 /opt/data

Описываем общие папки для публикации в конфигурационном файле /etc/samba/smb.conf:

vim /etc/samba/smb.conf

дописать 
[SAMBA]
		path = /opt/data
		writable = yes
		read only = no
		
valid users - это список пользователей, которым должно быть разрешено входить в эту службу 

systemctl restart samba

Проверяем

smbclient -L localhost -Uadministrator

Настройка сервера времени на SRV1-HQ на базе chrony

#Редактируем конфигурационный файл
vim /etc/chrony.conf

pool ntp2.vniiftri.ru iburst



allow ntp 
allow 192.168.11.0/24
**прописать остальныек свои сети 


#Перезапускаем службу chronyd
systemctl restart chronyd
 
#Проверяем статус служб chronyd
systemctl status chronyd

Настройка сервера времени на SRV1-HQ на базе chrony

#Проверяем с каким сервером 
синхронизировалось время
chronyc tracking

#Проверяем часовой пояс
timedatectl  

#Если часовой пояс отличается от московского
timedatectl  set-timezone Europe/Moscow





Настройка клиента сервера времени на ALT на базе chrony

#Редактируем конфигурационный файл
vim /etc/chrony.conf

pool *имя сервера*.au.domen iburst

#Перезапускаем службу chronyd
systemctl restart chronyd
 
#Проверяем статус служб chronyd
systemctl status chronyd




Настройка клиента сервера времени на RED OS на базе chrony

#Редактируем конфигурационный файл
vim /etc/chrony.conf

server *имя сервера*.au.team iburst

#Перезапускаем службу chronyd
systemctl restart chronyd
 
#Проверяем статус служб chronyd
systemctl status chronyd


Настройка клиента сервера времени на RED OS на базе chrony

#Проверяем с каким сервером 
синхронизировалось время
chronyc tracking

#Проверяем часовой пояс
timedatectl  

#Если часовой пояс отличается от московского
timedatectl  set-timezone Europe/Moscow
