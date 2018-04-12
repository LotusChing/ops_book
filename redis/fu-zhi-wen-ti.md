# 复制问题

## 复制建立前的常见错误

**1. Error condition on socket for SYNC: Connection refused**

连接拒绝，通常是Redis启动监听

复制状态信息
```
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:down
```

日志信息
```
13563:S 05 Jan 10:20:28.391 * SLAVE OF 127.0.0.1:6379 enabled (user request from 'id=2 addr=127.0.0.1:61099 fd=5 name= age=43 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=slaveof')
13563:S 05 Jan 10:20:29.022 * Connecting to MASTER 127.0.0.1:6379
13563:S 05 Jan 10:20:29.022 * MASTER <-> SLAVE sync started
13563:S 05 Jan 10:20:29.022 # Error condition on socket for SYNC: Connection refused
13563:S 05 Jan 10:20:30.024 * Connecting to MASTER 127.0.0.1:6379
13563:S 05 Jan 10:20:30.024 * MASTER <-> SLAVE sync started
13563:S 05 Jan 10:20:30.024 # Error condition on socket for SYNC: Connection refused
13563:S 05 Jan 10:20:31.026 * Connecting to MASTER 127.0.0.1:6379
13563:S 05 Jan 10:20:31.027 * MASTER <-> SLAVE sync started
13563:S 05 Jan 10:20:31.027 # Error condition on socket for SYNC: Connection refused
```

**2. Timeout connecting to the MASTER...**

连接超时通常是网络问题或者安全策略问题，检查基础网络和安全策略，例如：tcp warpper、iptables

复制状态信息
```
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:192.168.2.20
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:1
master_link_down_since_seconds:1509839486
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

日志信息
```
2.2 日志信息，通过日志可以看到slave会每隔60秒重连一遍，超时时间受repl-timeout参数影响
29930:S 05 Nov 07:45:38.318 * SLAVE OF 192.168.2.20:6379 enabled (user request from 'id=2 addr=127.0.0.1:36805 fd=6 name= age=14 idle=0 flags=N db=0
 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=slaveof')
29930:S 05 Nov 07:45:38.737 * Connecting to MASTER 192.168.2.20:6379
29930:S 05 Nov 07:45:38.737 * MASTER <-> SLAVE sync started
29930:S 05 Nov 07:46:39.366 # Timeout connecting to the MASTER...
29930:S 05 Nov 07:46:39.366 * Connecting to MASTER 192.168.2.20:6379
29930:S 05 Nov 07:46:39.366 * MASTER <-> SLAVE sync started
29930:S 05 Nov 07:47:41.009 # Timeout connecting to the MASTER...
29930:S 05 Nov 07:47:41.009 * Connecting to MASTER 192.168.2.20:6379
29930:S 05 Nov 07:47:41.009 * MASTER <-> SLAVE sync started
```

**3. MASTER aborted replication with an error: NOAUTH Authentication required.**

master端开启了密码认证，slave没有配置密码，需要配置密码才能同步`config set masterauth <master-password>`或者修改配置文件`masterauth <master-password>`
复制状态
```
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:1
master_link_down_since_seconds:1515133090
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

日志信息
```
16264:S 05 Jan 14:15:54.247 * SLAVE OF 127.0.0.1:6379 enabled (user request from 'id=2 addr=127.0.0.1:40060 fd=5 name= age=11 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=slaveof')
16264:S 05 Jan 14:15:54.541 * Connecting to MASTER 127.0.0.1:6379
16264:S 05 Jan 14:15:54.541 * MASTER <-> SLAVE sync started
16264:S 05 Jan 14:15:54.541 * Non blocking connect for SYNC fired the event.
16264:S 05 Jan 14:15:54.541 * Master replied to PING, replication can continue...
16264:S 05 Jan 14:15:54.541 * (Non critical) Master does not understand REPLCONF listening-port: -NOAUTH Authentication required.
16264:S 05 Jan 14:15:54.542 * (Non critical) Master does not understand REPLCONF capa: -NOAUTH Authentication required.
16264:S 05 Jan 14:15:54.542 * Partial resynchronization not possible (no cached master)
16264:S 05 Jan 14:15:54.542 # Unexpected reply to PSYNC from master: -NOAUTH Authentication required.
16264:S 05 Jan 14:15:54.542 * Retrying with SYNC...
16264:S 05 Jan 14:15:54.542 # MASTER aborted replication with an error: NOAUTH Authentication required.
16264:S 05 Jan 14:15:55.543 * Connecting to MASTER 127.0.0.1:6379
16264:S 05 Jan 14:15:55.543 * MASTER <-> SLAVE sync started
16264:S 05 Jan 14:15:55.543 * Non blocking connect for SYNC fired the event.
16264:S 05 Jan 14:15:55.543 * Master replied to PING, replication can continue...
16264:S 05 Jan 14:15:55.543 * (Non critical) Master does not understand REPLCONF listening-port: -NOAUTH Authentication required.
16264:S 05 Jan 14:15:55.543 * (Non critical) Master does not understand REPLCONF capa: -NOAUTH Authentication required.
16264:S 05 Jan 14:15:55.543 * Partial resynchronization not possible (no cached master)
16264:S 05 Jan 14:15:55.543 # Unexpected reply to PSYNC from master: -NOAUTH Authentication required.
```

## 复制建立后的常见问题
### 主从不一致是如何发生的？
* 常见于发生阻塞后主从数据不一致，通过监控来提前发现和处理

### redis是如何处理过期数据的，读到过期数据是什么情况？
redis处理过期数据有三种策略：
1. 惰性删除：当操作到key时，才会检查key有没有过期，如果过期才会删除掉这个key，这种策略对CPU很友好，有效节省CPU检测过期的开销，但是对内存不友好，如果冷数据过多会占用不必要的空间
2. 采样删除：惰性删除的问题在于不能及时的处理过期的冷数据，所以Redis会定期采样淘汰一批已过期的数据
3. 当前已用内存超过maxmemory限定时，触发主动清理策略
  * volatile-lru: 仅对设置过期的key做lru
  * allkeys-lru: 对所有key做lru
  * volatile-random: 随即删除即将过期的部分key
  * allkeys-random: 随即删除key
  * volatile-ttl: 删除即将过期的所有key
  * noeviction: 永不过期

避免读取到过期key可以通过配置，可以通过调高hz和采样大小maxmemory-samples，或者maxmemory-policy策略，PS: 这样会牺牲一些CPU性能

补充阅读：http://www.cnblogs.com/chenpingzhao/p/5022467.html


### 主从配置不一致
#### maxmemory
* 主从maxmemory配置不一致并不会导致主从无法进行，但是需要注意的是，如果slave数据超过maxmemory就会触发根据maxmemeory-policy策略进行清除过期数据，根据策略设置的不同有可能会导致清除掉有用的部分数据，而且，如果将来发送切主的时候可能会带来数据丢失的问题

#### 数据结构优化参数
例如hash-max-ziplist-entries等，如果master设置了，slave没设置或者master和slave不一致，则可能导致主从内存使用不一致的情况出现

### 规避全量复制

#### 什么时候会发生全量复制？
1. 主从第一次全量复制，无法避免
  * 设置maxmemory为小主节点，这样bgsave、rdb传输、加载都会很快，开销相对较小
  * 低峰执行
2. 当run id不匹配，slave会全量复制一遍master的数据
3. repl_back_buf复制缓冲区不足，offset在队列内则进行部分复制，超过缓冲队列则全量复制
  * 增大复制缓冲区大小，rel_backlog_size，复制缓冲区大小=(平均写入QPS*60)*故障分钟

### 复制风暴
#### 什么是复制风暴？
master重启后产生新的run id，所有slave发生全量复制，虽然rdb生成只执行一次，但是rdb传输会发生多次，整个过程对CPU、硬盘、网络都会带来开销

#### 单节点复制风暴
* 主节点重启，从全量复制

#### 单机器复制风暴
* 和上面情况类型，不过要对应的CPU、网络、磁盘开销要乘上所有主节点对应的从节点数
* 使用集群高可用，自动晋升slave为master

**使用标准化工具进行配置**