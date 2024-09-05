dnf install proftpd 
dnf install proftpd-utils

#Запуск сервиса proftpd и добавление его в автозагрузку

systemctl enable --now proftpd

добавьте параметр PROFTPD_OPTIONS="-DANONYMOUS_FTP" в файл /etc/sysconfig/proftpd. (в конец)
vi /etc/sysconfig/proftpd

#Добавляем ложную оболочку /sbin/nologin в файл списка оболочек /etc/shells.

echo /sbin/nologin >> /etc/shells

#Перезапускаем службу proftpd для применения настроек

systemctl restart proftpd

#При подключении анонимному пользователю доступны два каталога — pub и uploads, которые находятся в корневой директории FTP-сервера /var/ftp/.


#Определим другое расположение файлов для анонимного доступа по FTP изменив домашнюю директорию пользователя ftp

mkdir -p /srv/routers-conf 

chown ftp:ftp /srv/routers-conf
chmod 700 /srv/routers-conf

usermod -d /srv/routers-conf ftp


#Разрешим запись в корневой каталог анонимного пользователя

vi /etc/proftpd/anonftp.conf



Сохранение конфигурации маршрутизатора на базе EcoRouter
copy startup-config ftp ftp://192.168.33.2/27-06-2024_R-DT


Сохранение конфигурации маршрутизатора на базе vESR
confgure
archive
path ftp://192.168.33.2:/$H-$T
time-period 10080
auto
by-commit
exit


после чего необходимо сделать commit и confirm, чтобы сохранить конфигурацию



##*** 
time-period - это период времени в минутах, указывающий как часто нужно автоматически сохранять конфигурацию (независимо от изменений). В нашем случае - каждые 10080 минут (1 неделя).

auto - включает режим отправки файла конфигурации на сервер резервирования через указанный промежуток времени (промежуток задается через time-period).

by-commit - включает режим отправки файла конфигурации на сервер резервирования после удачного применения конфигурации (т.е. каждая команда commit применяет параметры и уже после их успешного применения сохраняет резервную копию конфигурации на FTP сервер).


