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

 pgbench -p 5433 homework6 -T  -R 100


homework6=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------
 0/605CEFD0
(1 row)


homework6=# SELECT checkpoints_timed, checkpoints_req FROM pg_stat_bgwriter;
 checkpoints_timed | checkpoints_req
-------------------+-----------------
                80 |               0
(1 row)


homework6=# SELECT pg_size_pretty('0/605CEFD0'::pg_lsn - '0/5827FB88'::pg_lsn);
 pg_size_pretty
----------------
 131 MB
(1 row)


```
Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

Выходит, что 16.8MB , но с такими раземерами min и  max wal не очень очевиден результат, уменшил размеры, но и по хорошему нужно подбирать размеры, и увеличивать время нагрузки, для получения более понятной информации.
```
ALTER SYSTEM SET min_wal_size = '16MB';
ALTER SYSTEM SET max_wal_size = '16MB';

SELECT pg_stat_reset_shared('bgwriter');
postgres=# SELECT pg_current_wal_insert_lsn();


 pg_current_wal_insert_lsn
---------------------------
 0/7CDE0830
(1 row)

пустил нагрузку



pgbench -p 5433 homework6 -T 200
pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 200 s
number of transactions actually processed: 142988
number of failed transactions: 0 (0.000%)
latency average = 1.399 ms
initial connection time = 6.122 ms
tps = 714.954639 (without initial connection time)


postgres=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------
 0/8F8E45A8
(1 row)

postgres=# SELECT checkpoints_timed, checkpoints_req FROM pg_stat_bgwriter;
 checkpoints_timed | checkpoints_req
-------------------+-----------------
                23 |              19
(1 row)

postgres=# SELECT pg_size_pretty('0/8F8E45A8'::pg_lsn - '0/7CDE0830'::pg_lsn);
 pg_size_pretty
----------------
 299 MB
(1 row)

postgres=# show fsync ;
 fsync
-------
 on
(1 row)

postgres=# show wal_sync_method ;
 wal_sync_method
-----------------
 fdatasync
(1 row)

alter system set fsync TO off ;

postgres@vaccuum001:/home/ubuntu$ pgbench -p 5433 homework6 -T 200
pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 200 s
number of transactions actually processed: 175004
number of failed transactions: 0 (0.000%)
latency average = 1.143 ms
initial connection time = 8.166 ms
tps = 875.054447 (without initial connection time)

Сменим 
alter system set synchronous_commit TO off ;


postgres@vaccuum001:/home/ubuntu$ pgbench -p 5433 homework6 -T 200
pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 200 s
number of transactions actually processed: 168993
number of failed transactions: 0 (0.000%)
latency average = 1.183 ms
initial connection time = 9.218 ms
tps = 844.999117 (without initial connection time)


alter system set wal_writer_delay TO  500;

pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 200 s
number of transactions actually processed: 157609
number of failed transactions: 0 (0.000%)
latency average = 1.269 ms
initial connection time = 6.291 ms
tps = 788.067728 (without initial connection time)


homework6=# alter system set synchronous_commit TO remote_write ;

```
Транзакций стало меньше, т.к. мы и увеличили задержку
