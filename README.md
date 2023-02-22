
# Тест Redis Sentinel

[Документация](https://redis.io/docs/management/sentinel/)

Запустить кластер:
```bash
$ docker compose up
```


## Общее по серверу Redis

Подключение к серверу:
```bash
$ docker exec -it redis-sentinel-redis-master-1 /bin/bash
$ redis-cli -p 6379
$ auth str0ng_passw0rd
```

[Переключение на другой протокол](https://redis.io/commands/hello/) (в зависимости от протокола меняется формат ответа):
```bash
$ hello
1# "server" => "redis"
2# "version" => "7.0.8"
3# "proto" => (integer) 3
4# "id" => (integer) 14
5# "mode" => "sentinel"
6# "modules" => (empty array)
```

Детальная информация по серверу:
```bash
$ info
...
```


## Действия с master сервером Redis

Добавить новую ноду реплики:
```bash
$ REPLICAOF master-host master-port
```

Состояние репликации:
```bash
$ info replication
# Replication
role:master
connected_slaves:2
slave0:ip=172.19.0.11,port=6379,state=online,offset=176365,lag=1
slave1:ip=172.19.0.12,port=6379,state=online,offset=176365,lag=1
master_failover_state:no-failover
master_replid:66ff8ffecd2378a0fe22cf5368f43b9bd64d82cd
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:176365
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:176365
```


## Тест переизбрания нового master

Отключить ноду `redis master`:
```bash
$ docker compose stop redis-master
```


## Действия с Sentinel

Войти в `Sentinel` (на самом деле в любой, они равноправны):
```bash
$ docker exec -it redis-sentinel-redis-sentinel-1 bash
$ redis-cli -p 6379
```

Параметры текущего `redis master`:
```bash
$ sentinel get-master-addr-by-name mymaster
1) "172.19.0.10"
2) "6379"
```

Информация о мастере `mymaster`:
```bash
$ sentinel master mymaster
 1# "name" => "mymaster"
 2# "ip" => "172.19.0.10"
 3# "port" => "6379"
 4# "runid" => "88860bf3630f37ee7af996b5715220ee2b435758"
 5# "flags" => "master"
 6# "link-pending-commands" => "0"
 7# "link-refcount" => "1"
 8# "last-ping-sent" => "0"
 9# "last-ok-ping-reply" => "757"
10# "last-ping-reply" => "757"
11# "down-after-milliseconds" => "10000"
12# "info-refresh" => "173"
13# "role-reported" => "master"
14# "role-reported-time" => "1459594"
15# "config-epoch" => "2"
16# "num-slaves" => "2"
17# "num-other-sentinels" => "2"
18# "quorum" => "2"
19# "failover-timeout" => "180000"
20# "parallel-syncs" => "1"
```

Инфомрация о репликах мастера `mymaster`:
```bash
$ sentinel replicas mymaster
1)  1# "name" => "172.19.0.12:6379"
    2# "ip" => "172.19.0.12"
    3# "port" => "6379"
    4# "runid" => "c71055730d8d8a821bd56f036d6896b824a24297"
    5# "flags" => "slave"
    6# "link-pending-commands" => "0"
    7# "link-refcount" => "1"
    8# "last-ping-sent" => "0"
    9# "last-ok-ping-reply" => "221"
   10# "last-ping-reply" => "221"
   11# "down-after-milliseconds" => "10000"
   12# "info-refresh" => "6664"
   13# "role-reported" => "slave"
   14# "role-reported-time" => "1482835"
   15# "master-link-down-time" => "0"
   16# "master-link-status" => "ok"
   17# "master-host" => "172.19.0.10"
   18# "master-port" => "6379"
   19# "slave-priority" => "100"
   20# "slave-repl-offset" => "357861"
   21# "replica-announced" => "1"
...
```

Переизбрание `redis-master`:
```bash
$ sentinel failover mymaster
```
