на EcoRouter

ntp server 192.168.11.2
ntp timezone utc+3
show ntp status
show ntp date
show ntp timezone


на vESR

ntp enable
ntp server 192.168.11.2
prefer    # Указываем предпочтительность данного NTP-сервера
minpoll 4 # Указываем интервал времени между отправкой сообщений NTP-серверу

exit

clock timezone gmt +3

do commit
do confirm

show ntp configuration
show ntp peers

set date 15:35:00 05 September 2024 # Еслри долго не синхрить, подогнать время поближе

