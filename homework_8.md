
ДЗ № 8  MVCC, vacuum и autovacuum.

После установки PostgreSQL в файле  postgresql.conf установили параметры настройки PostgreSQL 
из прикрепленного к материалам домашнего задания файла.
перезапустили PostgreSQL для применения настроек

Инициализировали тест производительности:
pgbench -i -U postgres postgres

Выполнили тест нагрузки
pgbench -c8 -P 60 -T 600 -U postgres postgres

с параметрами:
autovacuum_cost_limit = 5 
autovacuum_vacuum_cost_delay = 10
autovacuum_naptime = 1min

progress: 60.0 s, 1399.4 tps, lat 5.689 ms stddev 17.520
progress: 120.0 s, 1368.0 tps, lat 5.849 ms stddev 17.788
progress: 180.0 s, 909.9 tps, lat 8.752 ms stddev 25.330
progress: 240.0 s, 710.2 tps, lat 11.266 ms stddev 30.857
progress: 300.0 s, 464.0 tps, lat 17.233 ms stddev 44.240
progress: 360.0 s, 1091.4 tps, lat 7.331 ms stddev 21.060
progress: 420.0 s, 1087.2 tps, lat 7.358 ms stddev 20.656
progress: 480.0 s, 653.8 tps, lat 12.226 ms stddev 31.958
progress: 540.0 s, 450.4 tps, lat 17.765 ms stddev 44.840


уменьшили задержку выполнения autovacuum:
autovacuum_vacuum_cost_delay = 5

progress: 60.0 s, 1342.0 tps, lat 5.937 ms stddev 17.303
progress: 120.0 s, 1344.9 tps, lat 5.948 ms stddev 16.190
progress: 180.0 s, 1330.1 tps, lat 6.014 ms stddev 16.555
progress: 240.0 s, 1300.4 tps, lat 6.150 ms stddev 16.967
progress: 300.0 s, 1295.8 tps, lat 6.171 ms stddev 16.864
progress: 360.0 s, 1274.9 tps, lat 6.276 ms stddev 18.044
progress: 420.0 s, 1358.1 tps, lat 5.887 ms stddev 17.644
progress: 480.0 s, 1342.3 tps, lat 5.959 ms stddev 18.547
progress: 540.0 s, 1126.3 tps, lat 7.099 ms stddev 21.381
progress: 600.0 s, 1265.6 tps, lat 6.310 ms stddev 21.027

увеличили стоимость, а задержку выполнения вернули прежнию: 
autovacuum_cost_limit = 100
autovacuum_vacuum_cost_delay = 10

progress: 60.0 s, 1215.8 tps, lat 6.554 ms stddev 21.596
progress: 120.0 s, 1182.6 tps, lat 6.757 ms stddev 23.213
progress: 180.0 s, 1198.8 tps, lat 6.676 ms stddev 21.861
progress: 240.0 s, 1178.2 tps, lat 6.791 ms stddev 20.279
progress: 300.0 s, 1251.8 tps, lat 6.388 ms stddev 19.032
progress: 360.0 s, 1181.2 tps, lat 6.773 ms stddev 19.538
progress: 420.0 s, 1266.7 tps, lat 6.314 ms stddev 18.496
progress: 480.0 s, 1184.9 tps, lat 6.754 ms stddev 19.828
progress: 540.0 s, 1247.3 tps, lat 6.401 ms stddev 18.393
progress: 600.0 s, 1190.8 tps, lat 6.730 ms stddev 20.067


таким образом видно, что наиболее ровное значение tps получается 
или при увеличении стоимости очистки:
autovacuum_cost_limit = 100
autovacuum_vacuum_cost_delay = 10
или при уменьшении задержки очистки, что по моему предпочтительней(равномерней нагрузка):
vacuum_cost_limit = 5
autovacuum_vacuum_cost_delay = 5





