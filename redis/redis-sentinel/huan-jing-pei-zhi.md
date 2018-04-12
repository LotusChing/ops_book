# 环境配置

## 环境说明
1 Master
2 Slave
3 Sentinel

## 目录结构
```
[root@bj-vmware-test1 redis]# tree .
.
├── config
│   ├── redis_server_6379.conf
│   └── sentinel_26379.conf
├── data
│   ├── 26379.log
│   ├── 6379.aof
│   ├── 6379.log
│   └── 6379.rdb
└── run
    └── redis_6379.pid
```

## 配置文件

### Redis Server
**master**: 20
```
root@iZ947mgy3c5Z:/prodata/redis# cat config/redis_server_6379.conf 
daemonize yes
port 6379
bind 127.0.0.1
dbfilename "6379.rdb"
appendfilename "6379.aof"
dir "/prodata/redis/data"
logfile "/prodata/redis/data/6379.log"
pidfile "/prodata/redis/run/redis_6379.pid"
appendonly yes
```

**slave**: 30、50
```
daemonize yes
port 6379
bind 127.0.0.1
dbfilename "6379.rdb"
appendfilename "6379.aof"
dir "/prodata/redis/data"
logfile "/prodata/redis/data/6379.log"
pidfile "/prodata/redis/run/redis_6379.pid"
appendonly yes

slaveof 192.168.2.20 6379
```



