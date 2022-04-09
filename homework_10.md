ДЗ 10 Блокировки

Настроили вывод сообщений в журнал сервера:
log_lock_waits= true
deadlock_timeout =200 
log_min_duration_statement =200

открыли 3 подключения к нашей тестовой базе
psql -h 127.0.0.1 -U postgres -d test

В каждом подключении открыли транзакцию и поочереди 
выполнили команды update:
в 1-ом (pid=2812): update persons set first_name='f1' where id=1;
в 2-ом (pid=88):   update persons set first_name='f2' where id=1;
в 3-ем (pid=4932): update persons set first_name='f3' where id=1;

В 1-ом подключении команда выполнилась, а в остальных "зависли" ждут закрытия транзакции в 1-ом.
В это время в журнале можно увидеть записи о блокировках вида:
СООБЩЕНИЕ: процесс 88 продолжает ожидать в режиме ShareLock блокировку "транзакция 17560163" в течение 201.616 мс
....
СООБЩЕНИЕ: процесс 4932 продолжает ожидать в режиме ExclusiveLock блокировку "кортеж (0,2) отношения 30147 базы данных 30144" в течение 200.149 мс
...

pid  1 - 2812	2 - 88	3 - 4932

Получаем список блокировок (pg_locks):

pid	locktype	lockid		mode			granted	
88	"transactionid"	"17560163"	"ShareLock"		false
88	"relation"	"persons"	"RowExclusiveLock"	true
88	"tuple"		"persons:2"	"ExclusiveLock"		true
88	"transactionid"	"17560165"	"ExclusiveLock"		true
2812	"transactionid"	"17560163"	"ExclusiveLock"		true
2812	"relation"	"persons"	"RowExclusiveLock"	true
4932	"relation"	"persons"	"RowExclusiveLock"	true
4932	"transactionid"	"17560167"	"ExclusiveLock"		true
4932	"tuple"		"persons:2"	"ExclusiveLock"		false


По списку видно
в 1-ом соединении (pid=2812) есть блокировка таблицы (locktype="relation") и номер транзакции (lockid) 
во 2-ом соединении (pid=88)  цель блокировки (locktype="relation"), номер транзакции(lockid="17560165"), 
    номер транзакции закрытие которой мы ждем (lockid="17560163" granted=false) 
    и номер кортежа для этой транзакции. 
в 3-ем соединении (pid=49322) цель блокировки (locktype="relation"), номер транзакции(lockid="17560167"),
    номер кортежа c флагом granted=false т.е. с ожиданием закрытия предыдущей транзакции с таким же кортежем..

При выполнении команд update на всю таблицу (без where) из 2-х подключений
будет такая же ситуация команда из 2-го подключения будет ожидать когда закроется транзакция
по update из 1-го подключения.


transactionid 	Идентификатор транзакции, являющийся целью блокировки,
relation 	OID отношения, являющегося целью блокировки
tuple		Номер кортежа на странице, являющегося целью блокировки,

