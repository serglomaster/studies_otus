Домашнее задание №7

Логический уровень PostgreSQL

Зашли в созданный кластер под пользователем postgres

psql -h 127.0.0.1 -U postgres

Сооздали новую базу данных testdb:

create database testdb;

Вошли в базу данных под пользователем postgres:

psql -h 127.0.0.1 -U postgres -d testdb

Создали схему testnm

create schema testnm;

Создайли новую таблицу t1:

CREATE TABLE t1 (c1 int);

И заполнили ее данными:

INSERT INTO t1(c1) VALUES (1);

Новая роль readonly:

create role readonly;

Этой роли даем права на подключение к базе данных testdb и использование в ней
схемы testnm:

grant connect on database testdb to readonly;

grant usage on schema testnm to readonly;

и право на select для всех таблиц схемы testnm:

GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;

Создаем пользователя testread с паролем test123

CREATE USER testread WITH password 'test123';

Даем роль readonly пользователю testread:

GRANT readonly TO testread;

Переподключились к базе данных под пользователем testread:

psql -h 127.0.0.1 -U testread -d testdb

выполнили команду:

select \* from t1;

Команда вернула ошибку "нет доступа" так как таблица создана в схеме по
умолчанию public,

а права доступа к таблицам назначали на схему testnm.

Переподключились к базе данных под пользователем postgres:

Удалим таблицу t1

drop table t1;

Создаем ее заново но уже с явным указанием имени схемы testnm

CREATE TABLE testnm.t1 (c1 int);

И заполняем ее данными:

INSERT INTO testnm.t1(c1) VALUES (1);

Переподключились к базе данных под пользователем testread:

Команда select \* from testnm.t1;

вернула ошибку «нет доступа», так как права доступа на схему testnm были даны до
создания в ней таблицы t1

Надо снова дать права доступа к таблице t1 пользователю testread через роль
readonly.

GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;

Что бы эта ситуация не повторялась надо дать права доступа по умолчанию роли
readonly

ALTER default privileges in SCHEMA testnm grant SELECT on TABLEs to readonly;

Для всех новых таблиц будут назначаться права select для роли readonly.

выполняем команды:

create table t2(c1 integer); insert into t2 values (2);

команды выполнились без ошибки. Но мы опять создали таблицу в схеме public.
Поэтому и не было ошибок при создании.

Надо убрать права на схему public.

revoke CREATE on SCHEMA public FROM public;

revoke all on DATABASE testdb FROM public;

переподключились пользователем testread:

\\c testdb testread;

И команда

create table t3(c1 integer);

возвращает ошибку так как удалили права на схему public.
