Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
Установить на него PostgreSQL 15 с дефолтными настройками
Создать БД для тестов: выполнить pgbench -i postgres
Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
```
ubuntu@vaccuum001:~$ sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.

progress: 6.0 s, 1106.2 tps, lat 7.188 ms stddev 4.913, 0 failed

progress: 12.0 s, 1093.3 tps, lat 7.316 ms stddev 4.240, 0 failed
progress: 18.0 s, 1111.8 tps, lat 7.194 ms stddev 3.931, 0 failed
progress: 24.0 s, 1120.8 tps, lat 7.137 ms stddev 3.806, 0 failed
progress: 30.0 s, 1128.5 tps, lat 7.085 ms stddev 3.658, 0 failed
progress: 36.0 s, 1125.0 tps, lat 7.112 ms stddev 4.401, 0 failed
progress: 42.0 s, 1098.5 tps, lat 7.280 ms stddev 4.198, 0 failed
progress: 48.0 s, 1090.8 tps, lat 7.333 ms stddev 4.538, 0 failed
progress: 54.0 s, 1096.7 tps, lat 7.292 ms stddev 4.802, 0 failed
progress: 60.0 s, 1142.7 tps, lat 7.002 ms stddev 3.679, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 66694
number of failed transactions: 0 (0.000%)
latency average = 7.193 ms
latency stddev = 4.236 ms
initial connection time = 31.814 ms
tps = 1111.814027 (without initial connection time)
```
Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
Протестировать заново
Что изменилось и почему?
```
ubuntu@vaccuum001:~$ sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 1105.0 tps, lat 7.177 ms stddev 3.450, 0 failed
progress: 12.0 s, 1133.8 tps, lat 7.051 ms stddev 3.633, 0 failed
progress: 18.0 s, 1162.3 tps, lat 6.880 ms stddev 2.973, 0 failed
progress: 24.0 s, 1152.2 tps, lat 6.947 ms stddev 3.531, 0 failed
progress: 30.0 s, 1168.5 tps, lat 6.844 ms stddev 3.053, 0 failed
progress: 36.0 s, 1112.5 tps, lat 7.190 ms stddev 3.513, 0 failed
progress: 42.0 s, 1142.3 tps, lat 7.001 ms stddev 3.295, 0 failed
progress: 48.0 s, 1120.7 tps, lat 7.138 ms stddev 3.778, 0 failed
progress: 54.0 s, 1067.2 tps, lat 7.494 ms stddev 4.406, 0 failed
progress: 60.0 s, 1100.7 tps, lat 7.267 ms stddev 4.080, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 67599
number of failed transactions: 0 (0.000%)
latency average = 7.095 ms
latency stddev = 3.592 ms
initial connection time = 48.097 ms
tps = 1127.119915 (without initial connection time)
```
```

ubuntu@vaccuum001:~$ sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 991.8 tps, lat 8.002 ms stddev 3.505, 0 failed
progress: 12.0 s, 1184.0 tps, lat 6.756 ms stddev 3.046, 0 failed
progress: 18.0 s, 1189.7 tps, lat 6.724 ms stddev 2.964, 0 failed
progress: 24.0 s, 1195.5 tps, lat 6.687 ms stddev 3.053, 0 failed
progress: 30.0 s, 1194.5 tps, lat 6.698 ms stddev 2.910, 0 failed
progress: 36.0 s, 1185.8 tps, lat 6.745 ms stddev 3.046, 0 failed
progress: 42.0 s, 1098.2 tps, lat 7.281 ms stddev 4.766, 0 failed
progress: 48.0 s, 1184.8 tps, lat 6.753 ms stddev 3.011, 0 failed
progress: 54.0 s, 1190.0 tps, lat 6.723 ms stddev 3.000, 0 failed
progress: 60.0 s, 1158.2 tps, lat 6.906 ms stddev 3.336, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 69443
number of failed transactions: 0 (0.000%)
latency average = 6.908 ms
latency stddev = 3.313 ms
initial connection time = 42.620 ms
tps = 1157.655054 (without initial connection time)
```
честно говоря разницы в выводе почти нет)


Посмотреть размер файла с таблицей
```
679M    /var/lib/postgresql/15/main/base/5
postgres=# SELECT pg_size_pretty(pg_total_relation_size('homework5'));
 pg_size_pretty
----------------
 651 MB
(1 row)

```
после нескольких итерация
```
1.8G    /var/lib/postgresql/15/main/base/5
pg_size_pretty
----------------
 1799 MB
(1 row)
```
Даже после удаления таблиц 
```
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs
;
     relname      | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
------------------+------------+------------+--------+-------------------------------
 pgbench_tellers  |          0 |          0 |      0 |
 pgbench_history  |          0 |          0 |      0 |
 pgbench_accounts |          0 |          0 |      0 |
 homework5        |    9975634 |        184 |      0 | 2024-02-05 16:55:03.490892+03
 pgbench_branches |          0 |          0 |      0 |

postgres=# delete from homework5 where randomdata IS NOT NULL;
DELETE 10000002
1.8G    /var/lib/postgresql/15/main/base/5
postgres=# SELECT pg_size_pretty(pg_total_relation_size('homework5'));
 pg_size_pretty
----------------
 16 kB
(1 row)
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs
;
     relname      | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
------------------+------------+------------+--------+-------------------------------
 pgbench_tellers  |          0 |          0 |      0 |
 pgbench_history  |          0 |          0 |      0 |
 pgbench_accounts |          0 |          0 |      0 |
 homework5        |          0 |          0 |      0 | 2024-02-05 17:00:55.567647+03
 pgbench_branches |          0 |          0 |      0 |
(5 rows)

