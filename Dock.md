https://redos.red-soft.ru/base/arm/arm-other/docker-install/


#Установка Docker и Docker-compose
dnf install docker-ce
dnf install docker-ce-cli
dnf install docker-compose

#Включаем и добавляем службу в автозагрузки 
systemctl enable docker --now

#Прверяем статус службу
systemctl status docker

#По умолчанию доступ к среде контейнеризации и запуску сервисов имеет только суперпользователь.
#Для использования и управления средой контейнеризации обычным пользователем необходимо добавить группу docker.

usermod -aG docker <ИМЯ_ПОЛЬЗОВАТЕЛЯ>

#Основные команды Docker

docker build	Построить Docker-образ из Docker-файла.
docker images	Вывести список образов верхнего уровня.
docker image	Управление образами (docker image build - создание нового образа)
docker ps	Вывести список активных контейнеров.
docker start	Запустить контейнер.
docker stop	Остановить активный контейнер.
docker restart	Перезапустить контейнер/
docker tag	Создать тег (метку) образа, ссылающийся на существующий образ.
docker exec	Выполнить команду в активном контейнере.
docker rm	Удалить контейнер.
docker rmi	Удалить образ.

#Создание локального хранилища образов (Docker Registry).

#Поднимаем контейнер Docker с именем DockerRegistry из образа registry:2. 

docker run -d -p 5000:5000 --restart=always --name DockerRegistry registry:2

где
-p 5000:5000 – порт, на котором контейнер будет слушать сетевые запросы
--restart=always – параметр, который позволит контейнеру автоматически запускаться после перезагрузки сервера.

#Пишем Dockerfile для приложения web.
vi Dockerfile

#Развертываем приложение NGINX на базе Alpine
FROM nginx:alpine
#Заменяем дефолтную страницу nginx своей
RUN rm -rf /usr/share/nginx/html/*
COPY ./index.html /usr/share/nginx/html
#Добавляем конфигурационный файл для нашего web приложения
COPY ./web.conf /etc/nginx/conf.d/web.conf
#Указываем на необходимость открыть порт
EXPOSE 80
#Переводит Nginx «на передний план» (если этого не сделать, контейнер остановится сразу после запуска)
ENTRYPOINT ["nginx", "-g", "daemon off;"]

#Пишем Dockerfile для приложения web.

vi Dockerfile



##

FROM nginx:Alpine
RUN rm -rf /usr/share/nginx/html/*
COPY ./index.html /usr/share/nginx/html/ 
COPY ./web.conf /etc/nginx/conf.d/web.conf
EXPOSE 80
ENTRYPOINT ["nginx", "-g", "daemon off;"] 

#Создаем HTML страницу для web приложения

vi index.html

<html>
		<body>
				<h1>WEB_WSR</h1>
		</body>
</html>

#Создаем конфигурационный файл для web приложения

vi web.conf

server {
		listen 80;
		server_name www.au.team;
		location / {
				root /usr/share/nginx/html;
			}
}

server {
		listen 80;
		server_name www.au.team.ru;
		location / {
				root /usr/share/nginx/html;
			}
}

server {
		listen 80;
		server_name 192.168.33.3;
		return 404;
}

server {
		listen 80;
		server_name 172.16.4.14;
		return 404;
}


#Выполняем сборку образа

docker build -t web .

-t - позволяет присвоить имя собираемому образу;
"." - говорит о том что Dockerfile находится в текущей директории откуда выполняется данная команда и имеет имя именно Dockerfile

#Проверяем наличие собранного образа

docker images

#Присваиваем образу тег для размещения его в локальном Docker Registry

docker tag web localhost:5000/web:1.0

#Загружаем образ в локальный Docker Registry:

docker push localhost:5000/web:1.0

#Проверяем наличие образа локальном Docker Registry

docker images


#Удаляем образы localhost:5000/web:1.0 и web

docker rmi web
docker rmi localhost:5000/web:1.0

#Проверяем наличие образов

docker images

#Загружаем образ приложения web из локального Docker Registry:

docker pull localhost:5000/web:1.0

#Проверяем наличие образов

docker images

#Загружаем образ приложения web из локального Docker Registry:

docker pull localhost:5000/web:1.0

#Проверяем наличие образов

docker images


#Запускаем приложение web из скаченного образа из локального репозитория

docker run -d --name web -p 80:80 --restart=always localhost:5000/web:1.0

#Проверяем, что контейнер запустился

docker ps -a


проброс портов на маршрутизаторе для приложения web

Т.к. наше web приложение работает за NAT, необходимо настроить проброс портов на маршрутизаторе (R-DT)

Задаём правило для проброса порта на R-DT (EcoRouter)

При обращении на внешний адрес маршрутизатора (172.16.4.14) на порт 80 должен происходить проброс на адрес 192.168.33.3 (SRV2-DT) на порт 80 по протоколу tcp

R-DT(config)#ip nat source static tcp 192.168.33.3 80 172.16.4.14 80

#Проверяем работоспособность приложения web на внутреннем клиенте
На CLI необходимо прописать имя www.au.team.ru в файл /etc/hosts

#Переходим в браузер http://<IP адрес SRV2-DT>:3000
#Для доступа в веб-интерфейс Grafana - стандартный логин и пароль "admin"
#Задаём новый пароль (P@ssw0rd) и подтверждаем его


#Добавляем в Grafana – Prometheus

1) на главном меню нажимаем Add your first data source
2) выбираем Prometheus
3) вводим адрес контейнера с Prometheus
4) внизу на этой же странице нажимаем Save and Test


#Добавляем node-exporter в prometheus.yml расположенный в volumes docker

vim /var/lib/docker/volumes/root_prom-configs/_data/prometheus.yml

#Добавляем в конец файла информацию о контейнере с node-exporter

- job name: "prometheus"

- targets: ["localhost:9090"]

- job name: "node-exporter"

	static_configs:
		- targets: ["NodeExporter:9100"]


#Перезапускаем контейнеры

docker-compose -f monitoring.yml restart

#Переходим в веб-интерфейс prometheus по http://<IP адрес SRV2-DT>:9090

#Создаем требуемые Dashboard или скачиваем и устанавливаем подходящие шаблоны с сайта https://grafana.com/grafana/dashboards/

(например, шаблон Node Exporter Full – https://grafana.com/grafana/dashboards/12486-node-exporter-full/)


SRV1-DT  и SRV3-DT

Устанавливаем node-exporter:
dnf install prometheus-node_exporter -y
Запускаем службу 
systemctl enable --now prometheus-node_exporter
