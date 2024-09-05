#Установка WireGuard на RED OS

dnf install wireguard-tools


#Настройка WireGuard на сервере ()

#Необходимо сгенерировать на сервере публичные и закрытые ключи для wireguard

wg genkey | tee /etc/wireguard/privatekey | wg pubkey | tee /etc/wireguard/publickey

#Будет создано 2 файла - /etc/wireguard/privatekey и /etc/wireguard/publickey.

#Назначаем права доступа к файлу приватного (закрытого) ключа

chmod 600 /etc/wireguard/privatekey

#Настройка WireGuard на сервере ()

#Создаем ключи для клиента

wg genkey | tee /etc/wireguard/cli_privatekey | wg pubkey | tee /etc/wireguard/cli_publickey

#Настройка WireGuard на сервере ()

#Настраиваем конфигурационный файл интерфейса для wireguard. (В нашем случае, имя интерфейса будет wg0)

vi /etc/wireguard/wg0.conf



**где

[Interface]
PrivateKey - содержимое файла /etc/wireguard/privatekey
Address - адрес для vpn-сети (может быть изменен по вашему усмотрению)
ListenPort - порт, на котором работает WireGuard

[Peer]
PublicKey - содержимое файла /etc/wireguard/cli_publickey
AllowedIPs - разрешенный диапазон IP для клиентов


#Настройка WireGuard на сервере ()

#Добавляем в автозагрузку и запускаем WireGuard

systemctl enable --now wg-quick@wg0

#Проверяем статус WireGuard

systemctl status wg-quick@wg0

где wg0 - имя интефейса WireGuard


 наш VPN сервер работает за NAT, необходимо настроить проброс портов на маршрутизаторе ()

Задаём правило для проброса порта на (EcoRouter)

При обращении на внешний адрес маршрутизатора (172.16.4.14) на порт 51820 должен происходить проброс на адрес 192.168.33.4 (SRV3-DT) на порт 51820

(config)#ip nat source static udp 192.168.33.4 51820 172.16.4.14 51820


#Или пишем правила на фаерволе и включаем NAT на сервере, пример:

[Interface]
PrivateKey = *Твой ключ* ML****=* 
Address = 10.6.6.1/24
ListenPort = 51818
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o *tvoy_interface srv* -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o *tvoy_interface srv* -j MASQUERADE

[Peer]
PublicKey = *Tvoy_KLey* 
AllowedIPs = 10.6.6.2/32


#ИЛИ

На маршрутизаторе  добавляем статический маршрут до сети VPN

(config)#ip route 10.6.6.0/24 192.168.33.4

где 
10.6.6.0/24 – VPN сеть
192.168.33.4 – адрес сервера WireGuard


#Настройка WireGuard на внешнем клиенте (CLI)

#Устанавливаем wg-quick

apt-get install wireguard-tools-wg-quick

#Настраиваем конфигурационный файл интерфейса для wireguard. (В нашем случае, имя интерфейса будет wg0)

mkdir /etc/wireguard

vim /etc/wireguard/wg0.conf


где

[Interface]
PrivateKey - содержимое файла /etc/wireguard/cli_privatekey (создавали на SRV1-DT)
Address - адрес CLI в vpn-сети 
DNS - адрес DNS сервера клиента vpn-сети (не обязательно)

[Peer]
PublicKey - содержимое файла /etc/wireguard/publickey (создавали на SRV1-DT)
Endpoint = <SERVER_IP>:51820 (<SERVER_IP> - внешний адрес сервера)
AllowedIPs - разрешенный диапазон IP для клиентов
PersistentKeepalive - с какой периодичностью клиент будет отправлять пакет на сервер, чтобы обеспечит обновление данных об активных соединениях.


#Активируется wireguard-соединение командой:

wg-quick up wg0

Деактивируется wireguard-соединение командой:

wg-quick down wg0

где wg0 - имя интефейса WireGuard


На сервере сконфигурируйте VPN сервер
e) Запуск соединения осуществляется скриптом wg_connect, остановка wg_disconnect. 

https://www.altlinux.org/WireGuard
