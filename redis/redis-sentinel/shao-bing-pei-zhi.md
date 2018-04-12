# 哨兵配置

## 最简哨兵配置
```
root@iZ947mgy3c5Z:/prodata/redis# cat config/sentinel_26379.conf
port 26379
daemonize yes
dir "/prodata/redis/data"
logfile "26379.log"
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

## 配置参数说明
* sentinel monitor：配置主从的名称(标识)，以及主从架构中master的ip、端口，以及quorum(法定人数)，quorum这里配置的是2，代表大于等于2个sentinel节点认为宕机才会触发自动failover，否则不会进行failover
* sentinel down-after-milliseconds：sentinel检查redis实例超过指定的毫秒数才会认为是宕机，默认是30秒，sentinel每秒向监控的redis发送ping命令，如果redis实例返回的有效ping距离现在超过30秒，则认为该redis出现问题，则Sentinel将这个节点标记为主观下线(subjectively down, 简称SDown)，如果标记该redis实例的sentinel数超过quorum，则将该实例标记为客观下线(Objectively Down，简称ODown)，如果没超过quorum则清除该redis实例的主观下线标记
* sentinel parallel-syncs：当发生failover后，最多允许几个slave同时向master复制数据，当redis实例很多时，这个值越小，完成整个主从架构的故障转移需要的时间越长，但是如果这个值设置的很多，会导致大量从节点在接收和载入主节点数据时无法提供读请求的响应，所以默认建议设置1，只有一台处于重新复制新主状态无法服务，其余节点排队等待重新复制新主数据
* sentinel failover-timeout：如果整个failover过程超过指定毫秒，则认为本次failover失败


## 启动载入配置
20、30、50分别执行
```
root@iZ947mgy3c5Z:~# redis-sentinel /prodata/redis/config/sentinel_26379.conf
```