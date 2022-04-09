ДЗ 9 Журналы

Установили выполнение контрольной точки раз в 30 секунд.
checkpoint_timeout = 30s

Инициализировали тест производительности:
pgbench -i -U postgres postgres

Выполнили тест нагрузки
pgbench -c8 -P 60 -T 600 -U postgres postgres
За время тестирования сгенерировалось журнальных файлов общим объемом около 350Mb


Сравним tps в синхронном/асинхронном режиме:
тест в синхронном режиме:
ALTER SYSTEM SET synchronous_commit = on;
pgbench -P 1 -T 10 test

progress: 1.0 s, 1210.0 tps, lat 0.799 ms stddev 0.566
progress: 2.0 s, 1295.0 tps, lat 0.772 ms stddev 0.716
progress: 3.0 s, 1344.0 tps, lat 0.743 ms stddev 0.544
progress: 4.0 s, 1308.0 tps, lat 0.764 ms stddev 0.663
progress: 5.0 s, 1306.9 tps, lat 0.765 ms stddev 0.680
progress: 6.0 s, 1358.2 tps, lat 0.735 ms stddev 0.493
progress: 7.0 s, 1315.8 tps, lat 0.760 ms stddev 0.676
progress: 8.0 s, 1242.1 tps, lat 0.803 ms stddev 0.883
progress: 9.0 s, 1261.0 tps, lat 0.793 ms stddev 0.892
progress: 10.0 s, 1350.5 tps, lat 0.734 ms stddev 0.523

тест в асинхронном режиме:
ALTER SYSTEM SET synchronous_commit = off;
pgbench -P 1 -T 10 test

progress: 1.0 s, 1466.9 tps, lat 0.653 ms stddev 0.592
progress: 2.0 s, 1596.1 tps, lat 0.626 ms stddev 0.724
progress: 3.0 s, 1696.0 tps, lat 0.589 ms stddev 0.543
progress: 4.0 s, 1661.0 tps, lat 0.602 ms stddev 0.730
progress: 5.0 s, 1747.0 tps, lat 0.572 ms stddev 0.573
progress: 6.0 s, 1646.0 tps, lat 0.607 ms stddev 1.090
progress: 7.0 s, 1574.1 tps, lat 0.631 ms stddev 0.782
progress: 8.0 s, 1556.7 tps, lat 0.646 ms stddev 0.903
progress: 9.0 s, 1705.0 tps, lat 0.586 ms stddev 0.566
progress: 10.0 s, 1704.1 tps, lat 0.586 ms stddev 0.711

Видим, что скорость выполнения выросла т.к. не ждем сохранения записей WAL транзакции
для начала выполнения следующей.


включили контрольную сумму страниц для существующего кластера, предварительно остановив PostgreSQL
pg_checksums -e -D  /database

В базе test создаем таблицу test и наполняем ее данными:
create database test;
CREATE TABLE test(i int, first_name text);
insert into test (id, first_name) values(1, 'sergeev'), (2,'ivanov');

Определим в каком файле находятся данные по этой таблице
SELECT pg_relation_filepath('test');   
это файл "base/30144/30147" 
Остановили сервер, нашли этот файл и изменили в нем любой байт(сделал это текстовым редактором)
Запустили сервер.
При попытке чтения данных из этой таблицы нам возвращается ошибка, изменилась контрольная сумма
файла.
Что бы восстановить возможность работы хоть и с потерей данных надо выставить параметр:
ignore_checksum_failure=on  в postgresql.conf
перезапустили сервер для применения настроек
выполнили select * from test;
команды выполнилась без ошибок но данных в таблице нет.
Таким образом восстановили работоспособность таблицы хотя и с потерей в ней данных. 


