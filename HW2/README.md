## Сравнение производительности при 500+ коннектах



## Конфигурация стенда
- Ванильный PostgreSQL
- pgbouncer
- pgcat
- haproxy

- Виртуальная машина 8 RAM 4 CPU
- Almalinux 9
- PostgreSQL 17
- БД "тайские перевозки"



## На сервере произведены настройки

#### PostgreSQL
```
max_connections = 600
# Add settings for extensions here
shared_buffers = 2GB
work_mem = 32MB
maintenance_work_mem = 400MB
effective_cache_size = 6GB
effective_io_concurrency = 100
random_page_cost = 1.1
#max_worker_processes = 8
#max_parallel_workers_per_gather = 2
#max_parallel_maintenance_workers = 2
#max_parallel_workers = 4
commit_delay = 0
# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '1024 MB'
min_wal_size = '512 MB'
# WAL writing
wal_compression = on
wal_buffers = -1
# auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB
# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0
# Parallel queries:
max_worker_processes = 4
max_parallel_workers_per_gather = 2
max_parallel_maintenance_workers = 2
max_parallel_workers = 4
parallel_leader_participation = on
# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 100
wal_recycle = on
```

#### pgbouncer
```
max_client_conn = 2000
default_pool_size = 200

```

#### pgcat
```
[general]
port = 7432
admin_username = "pgcat"
admin_password = "my-pony-likes-to-dance-tango"
worker_threads = 10
connect_timeout = 5000

[pools.thai]

[pools.thai.users.0]
pool_size = 20
username = "thai_user"
password = "PAHv7mt4"

[pools.thai.shards.0]
database = "thai"
servers = [
    ["127.0.0.1", 5432, "primary"],
]
```



## Тесты

### Ванильный PostgreSQL + настройки производительности
```
# pgbench -c 500 -j 4 -T 60 -f ~/workload.sql -U thai_user -h 192.168.31.191 -p 5432 -d thai -n
Password:
pgbench (17.2)
transaction type: /root/workload.sql
scaling factor: 1
query mode: simple
number of clients: 500
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 207101
number of failed transactions: 0 (0.000%)
latency average = 137.731 ms
initial connection time = 3821.285 ms
tps = 3630.259552 (without initial connection time)

# pgbench -c 600 -j 4 -T 60 -f ~/workload.sql -U thai_user -h 192.168.31.191 -p 5432 -d thai -n
Password:
pgbench (17.2)
pgbench: error: connection to server at "192.168.31.191", port 5432 failed: FATAL:  remaining connection slots are reserved for roles with the SUPERUSER attribute
pgbench: error: could not create connection for client 297
```

### pgbouncer
```
# pgbench -c 500 -j 4 -T 60 -f ~/workload.sql -U thai_user -h 192.168.31.191 -p 6432 -d thai -n
Password:
pgbench (17.2)
transaction type: /root/workload.sql
scaling factor: 1
query mode: simple
number of clients: 500
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 145483
number of failed transactions: 0 (0.000%)
latency average = 194.465 ms
initial connection time = 3673.275 ms
tps = 2571.163225 (without initial connection time)

# pgbench -c 1000 -j 4 -T 60 -f ~/workload.sql -U thai_user -h 192.168.31.191 -p 6432 -d thai -n
Password:
pgbench (17.2)
transaction type: /root/workload.sql
scaling factor: 1
query mode: simple
number of clients: 1000
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 110211
number of failed transactions: 0 (0.000%)
latency average = 496.688 ms
initial connection time = 7701.970 ms
tps = 2013.335353 (without initial connection time)
```


### pgcat
```
[root@MiWiFi-RC06-srv ~]# pgbench -c 500 -j 4 -T 60 -f ~/workload.sql -U thai_user -h 192.168.31.191 -p 7432 -d thai -n
Password:
pgbench (17.2)
transaction type: /root/workload.sql
scaling factor: 1
query mode: simple
number of clients: 500
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 151018
number of failed transactions: 0 (0.000%)
latency average = 195.846 ms
initial connection time = 1162.270 ms
tps = 2553.022320 (without initial connection time)

# pgbench -c 1000 -j 4 -T 60 -f ~/workload.sql -U thai_user -h 192.168.31.191 -p 7432 -d thai -n
Password:
pgbench (17.2)
transaction type: /root/workload.sql
scaling factor: 1
query mode: simple
number of clients: 1000
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 154134
number of failed transactions: 0 (0.000%)
latency average = 378.518 ms
initial connection time = 2260.695 ms
tps = 2641.885703 (without initial connection time)
```
### haproxy + pgcat
```
# pgbench -c 1000 -j 4 -T 60 -f ~/workload.sql -U thai_user -h 192.168.31.191 -p 8432 -d thai -n
Password:
pgbench (17.2)
transaction type: /root/workload.sql
scaling factor: 1
query mode: simple
number of clients: 1000
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 180050
number of failed transactions: 0 (0.000%)
latency average = 322.782 ms
initial connection time = 2415.379 ms
tps = 3098.067365 (without initial connection time)
```

## Результаты и выводы
### postgresql
При приближениии количества клиентов к максимально разрешенному PostgreSQL обрывает конекты.
### pgbouncer
Для решения проблемы с максимальным количеством открытых файлов pgbouncer был увеличин limit открытых файлов через директиву LimitNOFILE=8192 в описании systemd units.
### pgcat
В стандартных настройках, в рамках тестирования, не укладывается в таймауты ожидания свободных конектов к PostgreSQL для этого было увеличено количество рабочих воркеров(worker_threads) до 10 потоков и параметр ожидания конекта(connect_timeout) до 5 секунд как результат увеличение нагрузки на ЦПУ.


По результатам тестов видно что pgcat выдает большее количество tps и меньшее время задержки с меньшим временем инициации конектов. Но так как проект ведет активную разработку много открытых issues которые мешают в работе и перед внедрением в продуктовой среде требуется более глубокое тестирование.


