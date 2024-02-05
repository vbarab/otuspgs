Настройте выполнение контрольной точки раз в 30 секунд.
```
alter system set checkpoint_timeout = '30s';
SELECT pg_reload_conf();

postgres=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------
 0/25DB5D0
(1 row)

postgres@vaccuum001:/home/ubuntu$ pgbench -p 5433 homework6 -T 600
pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.

postgres@vaccuum001:/home/ubuntu$ pgbench -p 5433 homework6 -T 600
pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 462159
number of failed transactions: 0 (0.000%)
latency average = 1.298 ms
initial connection time = 5.760 ms
tps = 770.272084 (without initial connection time)




You are now connected to database "homework6" as user "postgres".
homework6=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------
 0/5827FB88
(1 row)

 pgbench -p 5433 homework6 -T 10 -R 100



```
Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

Выходит, что 16.8MB 
