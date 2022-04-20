ДЗ 12  Настройка PostgreSQL 

Для выполнения ДЗ использовалась виртуальная машина Yandex RAM 2G CPU 2.

Инициализировали тест производительности:
pgbench -i -U postgres postgres

Выполнили тест нагрузки
pgbench -c 50 -j 2 -P 10 -T 60 -U postgres postgres

Для только установленного PostgreSQL на виртуальной машине  
результат теста нагрузки:
progress: 10.0 s, 184.6 tps, lat 235.884 ms stddev 257.556
progress: 20.0 s, 153.8 tps, lat 326.197 ms stddev 325.677
progress: 30.0 s, 145.6 tps, lat 341.549 ms stddev 320.590
progress: 40.0 s, 136.4 tps, lat 372.156 ms stddev 380.094
progress: 50.0 s, 142.8 tps, lat 346.826 ms stddev 343.082
progress: 60.0 s, 166.8 tps, lat 300.944 ms stddev 300.927

Изменили параметры сервера (postgresql.conf):
Выделили серверу больше памяти:  			   shared_buffers = 512MB
Максимальное количество памяти для буферизации данных WAL: wal_buffers = 16MB
Максимальный объём памяти при обработке запросов: 	   work_mem = 2621kB
Объём памяти для операций обслуживания БД:		   maintenance_work_mem = 128MB
Уменьшим стоимость чтения произвольной страницы с диска:   random_page_cost = 1.1

Изменили на эти значения параметры в postgresql.conf и перезапустили PostgreSQL
sudo pg_ctlcluster 14 main restart

результат теста нагрузки:
progress: 10.0 s, 639.7 tps, lat 75.330 ms stddev 91.165
progress: 20.0 s, 461.3 tps, lat 107.189 ms stddev 144.939
progress: 30.0 s, 430.4 tps, lat 117.901 ms stddev 178.490
progress: 40.0 s, 495.4 tps, lat 100.930 ms stddev 133.054
progress: 50.0 s, 535.4 tps, lat 93.110 ms stddev 116.265
progress: 60.0 s, 485.6 tps, lat 101.473 ms stddev 126.875

Как видим произошло увеличение производительности.

Изменим еще один параметр
Включим асинхронное подтверждение транзакций: synchronous_commit = off
 
результат теста нагрузки:
progress: 10.0 s, 1661.0 tps, lat 29.003 ms stddev 23.690
progress: 20.0 s, 1792.2 tps, lat 27.828 ms stddev 23.026
progress: 30.0 s, 1714.5 tps, lat 29.066 ms stddev 23.560
progress: 40.0 s, 1672.0 tps, lat 29.794 ms stddev 24.498
progress: 50.0 s, 1665.7 tps, lat 29.882 ms stddev 24.314
progress: 60.0 s, 1615.2 tps, lat 30.927 ms stddev 25.829

Произошло значительное увеличение производительности, хотя и сриском потери данных
из за включенного асинхронного подтверждения транзакций.
