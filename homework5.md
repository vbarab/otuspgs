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
