Домашнее задание №6

Физический уровень PostgreSQL

На виртуальной машине (Yandex Cloud) установили PostgreSQL 14

sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo
sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt \$(lsb_release
\-cs)-pgdg main" \> /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O -
https://www.postgresql.org/media/keys/ACCC4CF8.asc \| sudo apt-key add - && sudo
apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install
postgresql-14

Создана тестовая база Test с таблицей test и заполнена произвольными данными

create database test;

CREATE TABLE test (i serial, amount int);

INSERT INTO test(amount) VALUES (100);

INSERT INTO test(amount) VALUES (500);

Остановили PostgreSQL

sudo -u postgres pg_ctlcluster 14 main stop

В Yandex Cloud создан новый диск и присоединен к виртуальной машине

Инициализация диска в виртуальной машине:

Утилита управления дисками: sudo apt-get install parted

Размечаем диск: sudo parted /dev/vdb mklabel gpt

Форматируем: sudo mkfs.ext4 -L datapartition /dev/vdb1

Создаем каталог: /mnt/data

Монтируем в него диск: sudo mount -o defaults /dev/vdb /mnt/data

Пользователя postgres сделали владельцем: chown -R postgres:postgres /mnt/data/

Перенесли содержимое: mv /var/lib/postgresql/14 /mnt/data

Попытались запустить PostgreSQL

Получили ошибку каталог /var/lib/postgresql/14/main не найден (что правильно
ведь его перенесли в другой)

Отредактировали в файле /etc/postgresql/14/main/postgresql.conf

параметр data_directory= ‘/mnt/data/14/main’

т.е. прописали новый путь к файлам кластера постгреса.

После этого запуск кластера прошел без ошибок.

sudo -u postgres pg_ctlcluster 14 main start

И подключившись к базе test можем прочитать наши тестовые данные из таблицы
test.

psql -h 127.0.0.1 -U postgres -d test

select \* from test;
