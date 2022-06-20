Секционирование таблицы:
Есть таблица с логом работы некоторого оборудования:
CREATE TABLE workdevice (
	id bigserial,
       	log_date    timestamptz,
	description text
   ) PARTITION BY RANGE(log_date);

Секционируем эту таблицу по суткам, создадим отдельные суточные секции:
create table workdevice_20220501 partition of workdevice for values from ('2022-05-01 00:00:00+03') to ('2022-05-02 00:00:00+03');
create table workdevice_20220502 partition of workdevice for values from ('2022-05-02 00:00:00+03') to ('2022-05-03 00:00:00+03');
create table workdevice_20220503 partition of workdevice for values from ('2022-05-03 00:00:00+03') to ('2022-05-04 00:00:00+03');

Для рабочего варианта надо реализовать в конце каждых суток выполнение по расписанию (cron) Функции/скрипта,
создающего секцию на следующие сутки.

Заполняем данными:

insert into workdevice (log_date, description) values ('2022-05-01 11:26:00','start work');
insert into workdevice (log_date, description) values ('2022-05-02 19:11:00','error work');
insert into workdevice (log_date, description) values ('2022-05-03 01:54:00','stop work');
Получаем в каждой секции по одной записи.

При команде вставить запись с датой по которой нет секции получим ошибку вставки
insert into workdevice (log_date, description) values ('2022-05-04 21:04:00','work');

