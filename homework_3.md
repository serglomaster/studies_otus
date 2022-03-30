Домашнее задание №3

Установка PostgreSQL

Работа выполнена в Yandex Cloud.

Для внешнего доступа к виртуальной машине использовалась программа PuTTy.

Устанавливаем сам Docker на виртуальную машину:

добавим ключ для работы с репозиторием Docker Hub:

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \| sudo apt-key add -

Добавим этот репозиторий в локальный список репозиториев

sudo add-apt-repository "deb [arch=amd64]
https://download.docker.com/linux/ubuntu \$(lsb_release -cs) stable"

Устанавливаем Docker:

sudo apt install docker-ce

Запустим демон Docker и активируем его автозапуск:

sudo systemctl start docker

sudo systemctl enable docker

получим из репозитория Docker Hub готовый образ контейнера с PostgreSQL:

sudo docker pull postgres

Создаем docker-сеть:

sudo docker network create pg-net

Создаем локально каталог /var/lib/postgres

Запускаем контейнер с образом postgres и с созданной сетью с монтированием
локального каталога с каталогом внутри контейнера:

sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres
\-d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

Редактируем сетевые правила подключения к PostgreSQL

В файле /var/lib/postgres/postgresql.conf

строка: listen_addresses = '\*'

В файле /var/lib/postgres/pg_hba.conf

добавим строку: host all all 0.0.0.0/0 md5

Перезагружаем контейнер для вступления изменений в силу.

docker restart pg-docker

Запускаем отдельный контейнер с клиентом в общей сети с БД:

sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h
pg-docker -U postgres

и в нем создаем базу test

create database test

в ней таблицу test

CREATE TABLE test (i serial, amount int);

и добавим строки в нее

INSERT INTO test(amount) VALUES (100);

INSERT INTO test(amount) VALUES (500);

Подключаемся с локальной машины по публичному IP к PostgreSQL

psql -h 51.250.106.183 -U postgres -d test;

Прочитали содержимое таблицы:

select \* from test;

Удаляем контейнер:

Остановили

docker stop pg-docker

Удалили

docker rm pg-docker

Создаем, запускаем заново контейнер PostgreSQL:

sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres
\-d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

так как все данные контейнера с PostgreSQL были на локальном каталоге то после
удаления контейнера они не удалились.

И при создании контейнера останутся прежние данные: настройки PostgreSQL и базы
с данными.

Опять подключаемся через контейнер с клиентом

выполняем select \* from test; (база test)

и видим все строки добавленные до удаления контейнера с PostgreSQL

Подключаемся с локальной машины по публичному IP к PostgreSQL

и по запросу видим те же данные.
