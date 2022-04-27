ДЗ 14  Репликации PostgreSQL 

Для выполнения ДЗ использовалась виртуальная машина Yandex RAM 2G CPU 2.
Для организации репликаций использовалось 3 кластера:
кластер № 1 port=5432
кластер № 2 port=5433
кластер № 3 port=5434

Для простоты у всех кластеров для суперпользователя установлен одинаковый пароль.

кластер № 1
Создаем базу:
create database name;	создать базу

таблица для публикации (с заполнением):
create table student as 
select  generate_series(1,10) as id, md5(random()::text)::char(10) as fio;

таблица для подписки:
create table student2 (id int, fio character(10));

Устанавливаем логическую репликацию (postgresql.conf):
wal_level = logical

И доступ для репликации (pg_hba.conf):
host	replication		postgres		0.0.0.0/16		md5

Доступ с любого IP суперпользователем, что конечно не правильно с точки зрения безопасности.
Надо создавать отдельного пользователя и прописывать конкретные адреса.
перезапускаем кластер
sudo pg_ctlcluster 14 main restart


кластер № 2
Создаем кластер
sudo pg_createcluster -d /var/lib/postgresql/14/main2 14 main2
И также как для 1-го кластера меняем настройки
Устанавливаем логическую репликацию (postgresql.conf):
wal_level = logical

И доступ для репликации (pg_hba.conf):
host	replication		postgres		0.0.0.0/16		md5

перезапускаем кластер
sudo pg_ctlcluster 14 main2 restart

Создаем базу:
create database name;

в ней таблица для публикации (с заполнением):
create table student2 as 
select  generate_series(1,10) as id, md5(random()::text)::char(10) as fio;

таблица для подписки:
create table student (id int, fio character(10));


Таблицы подготовили, теперь настраиваем репликацию:
кластер № 1
создаем публикацию на таблицу student:
CREATE PUBLICATION test_pub FOR TABLE student;

кластер № 2
создаем публикацию на таблицу student2:
CREATE PUBLICATION test_pub2 FOR TABLE student2;

кластер № 1
создадим подписку к базе на кластер №2 
CREATE SUBSCRIPTION test_sub 
CONNECTION 'host=127.0.0.1 port=5433 user=postgres password=pas123 dbname=name' 
PUBLICATION test_pub2 WITH (copy_data = true);

кластер № 2
создадим подписку к базе на кластер №1 
CREATE SUBSCRIPTION test_sub2 
CONNECTION 'host=127.0.0.1 port=5432 user=postgres password=pas123 dbname=name' 
PUBLICATION test_pub WITH (copy_data = true);

И можно посмотреть на обоих кластерах 
публикации и подписки
\dRp+
\dRs

кластер № 3
создаем 2 таблицы для подписок 
create table student (id int, fio character(10));
create table student2 (id int, fio character(10));

создадим подписку на кластер №1 к публикации таблицы studens 
CREATE SUBSCRIPTION test_sub3 
CONNECTION 'host=127.0.0.1 port=5432 user=postgres password=pas123 dbname=name' 
PUBLICATION test_pub WITH (copy_data = true);

создадим подписку на кластер №1 к публикации таблицы studens2 
CREATE SUBSCRIPTION test_sub4 
CONNECTION 'host=127.0.0.1 port=5432 user=postgres password=pas123 dbname=name' 
PUBLICATION test_pub2 WITH (copy_data = true);

Таким образом после всех настроек репликаций получились следующие цепочки передачи данных между таблицами:
таблица student кластер №1 одновременно реплицирует данные в таблицы кластеров 2 и 3.
таблица student2 кластер №2 реплицирует данные в таблицу кластера 1 а потом из кластера 1 в кластер 3.
Получается "цепочка" передачи данных по таблице student2.



