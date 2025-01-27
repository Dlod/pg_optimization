# HW1. Первичная настройка ОС и PostgreSQL.
## Конфигурация инстанса
cpu 4  
ram 8Gb  
disk 50Gb  
## Инициализируем тестовую базу данных
Создаем тестовую базу данных и создаем в ней записи, данных в 50 раз больше размера по умолчанию
```
psql -c "create database test_db
pgbench -i -s 50 test_db
```
## Параметры pgbench которые будем использовать
-c (клиенты), количество "клиентов", с которыми нужно соединиться, в примере открывает 10 одновременных сессий  
-j (threads), определяет количество рабочих процессов, в примере 2  
-t (транзакции), количество транзакций, которые нужно выполнить, в примере каждая сессия выполнит 10000 транзакций  
-r вывод отчета  
## Тестовый запуск с настройками по умолчанию 
```
[postgres@MiWiFi-RC06-srv ~]$ pgbench -r -c 10 -j 2 -t 10000 test_db
pgbench (17.2)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
maximum number of tries: 1
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
number of failed transactions: 0 (0.000%)
latency average = 7.513 ms
initial connection time = 19.700 ms
tps = 1330.958761 (without initial connection time)
statement latencies in milliseconds and failures:
         0.003           0  \set aid random(1, 100000 * :scale)
         0.001           0  \set bid random(1, 1 * :scale)
         0.001           0  \set tid random(1, 10 * :scale)
         0.001           0  \set delta random(-5000, 5000)
         0.442           0  BEGIN;
         0.649           0  UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
         0.567           0  SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
         0.614           0  UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
         0.866           0  UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
         0.534           0  INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
         3.470           0  END;
```
## Измененяем настройки работы с памятью
shared_buffers = 3GB  
work_mem = 24MB  
effective_cache_size = 6GB  
maintenance_work_mem = 400MB  
effective_io_concurrency = 100  
random_page_cost = 1.1  
commit_delay = 0  
wal_buffers = 64MB  

```
[postgres@MiWiFi-RC06-srv ~]$ pgbench -r -c 10 -j 2 -t 10000 test_db
pgbench (17.2)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
maximum number of tries: 1
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
number of failed transactions: 0 (0.000%)
latency average = 5.863 ms
initial connection time = 24.435 ms
tps = 1705.477769 (without initial connection time)
statement latencies in milliseconds and failures:
         0.002           0  \set aid random(1, 100000 * :scale)
         0.001           0  \set bid random(1, 1 * :scale)
         0.001           0  \set tid random(1, 10 * :scale)
         0.001           0  \set delta random(-5000, 5000)
         0.354           0  BEGIN;
         0.554           0  UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
         0.485           0  SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
         0.528           0  UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
         0.684           0  UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
         0.453           0  INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
         2.518           0  END;
```
## Изменяем настройки работы с ЦПУ
max_worker_processes = 4  
max_parallel_workers_per_gather = 2  
max_parallel_maintenance_workers = 2  
max_parallel_workers = 4  
```
[postgres@MiWiFi-RC06-srv ~]$ pgbench -r -c 10 -j 2 -t 10000 test_db
pgbench (17.2)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
maximum number of tries: 1
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
number of failed transactions: 0 (0.000%)
latency average = 5.455 ms
initial connection time = 20.779 ms
tps = 1833.267375 (without initial connection time)
statement latencies in milliseconds and failures:
         0.002           0  \set aid random(1, 100000 * :scale)
         0.001           0  \set bid random(1, 1 * :scale)
         0.001           0  \set tid random(1, 10 * :scale)
         0.001           0  \set delta random(-5000, 5000)
         0.322           0  BEGIN;
         0.498           0  UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
         0.438           0  SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
         0.476           0  UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
         0.622           0  UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
         0.408           0  INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
         2.401           0  END;
```




## Дальнейшие правки несут за собой возможную потерю данных в случае краха сервера
```
fsync = off
synchronous_commit = off
full_page_writes = off

[postgres@MiWiFi-RC06-srv ~]$ pgbench -r -c 10 -j 2 -t 10000 test_db
pgbench (17.2)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
maximum number of tries: 1
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
number of failed transactions: 0 (0.000%)
latency average = 3.220 ms
initial connection time = 24.716 ms
tps = 3105.167007 (without initial connection time)
statement latencies in milliseconds and failures:
         0.001           0  \set aid random(1, 100000 * :scale)
         0.001           0  \set bid random(1, 1 * :scale)
         0.001           0  \set tid random(1, 10 * :scale)
         0.000           0  \set delta random(-5000, 5000)
         0.305           0  BEGIN;
         0.420           0  UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
         0.381           0  SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
         0.433           0  UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
         0.560           0  UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
         0.370           0  INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
         0.317           0  END;

```



## Выводы
 - Изменение настроек работы с памятью дает существенный прирост в tps
 - Изменение настроек количества рабочих процессов и их параллельность так же увеличивают tps и сокращают задержки
 - Изменение параметров уменьшающих надежность PostgreSQL ускоряют работу в разы
