# 复制配置

配置复制支持三种方式

* 配置文件 slaveof master_ip master_port
* redis-server启动时添加--slaveof master_ip master_port
* 使用命令 slaveof master_ip master_port

最常用的就是1、3了

## 建立复制

master端：

```
root@iZ947mgy3c5Z:/opt/redis# redis-cli 
127.0.0.1:6379> keys *
1) "age"
```

slave端：执行SLAVEOF 127.0.0.1 6379后，本地的数据就被清空了，然后把master的数据拿了过来
```
root@iZ947mgy3c5Z:/opt/redis# redis-cli -p 6380
127.0.0.1:6380> keys *
(empty list or set)
127.0.0.1:6380> set age 10
OK
127.0.0.1:6380> set name da
OK
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379
OK
127.0.0.1:6380> keys *
1) "age"
```

master端日志：
```
32063:M 04 Jan 16:30:40.966 * Slave 127.0.0.1:6380 asks for synchronization
32063:M 04 Jan 16:30:40.966 * Full resync requested by slave 127.0.0.1:6380
32063:M 04 Jan 16:30:40.966 * Starting BGSAVE for SYNC with target: disk
32063:M 04 Jan 16:30:40.967 * Background saving started by pid 1796
1796:C 04 Jan 16:30:40.977 * DB saved on disk
1796:C 04 Jan 16:30:40.977 * RDB: 0 MB of memory used by copy-on-write
32063:M 04 Jan 16:30:41.034 * Background saving terminated with success
32063:M 04 Jan 16:30:41.035 * Synchronization with slave 127.0.0.1:6380 succeeded
```


slave端日志：
```
3623:S 04 Jan 16:30:40.354 * SLAVE OF 127.0.0.1:6379 enabled (user request from 'id=5 addr=127.0.0.1:25116 fd=5 name= age=49 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=slaveof')
3623:S 04 Jan 16:30:40.965 * Connecting to MASTER 127.0.0.1:6379
3623:S 04 Jan 16:30:40.966 * MASTER <-> SLAVE sync started
3623:S 04 Jan 16:30:40.966 * Non blocking connect for SYNC fired the event.
3623:S 04 Jan 16:30:40.966 * Master replied to PING, replication can continue...
3623:S 04 Jan 16:30:40.966 * Partial resynchronization not possible (no cached master)
3623:S 04 Jan 16:30:40.967 * Full resync from master: 81c3acbacf5fa0f01ecd48c285408c490e48a660:1
3623:S 04 Jan 16:30:41.034 * MASTER <-> SLAVE sync: receiving 28 bytes from master
3623:S 04 Jan 16:30:41.035 * MASTER <-> SLAVE sync: Flushing old data
3623:S 04 Jan 16:30:41.035 * MASTER <-> SLAVE sync: Loading DB in memory
3623:S 04 Jan 16:30:41.035 * MASTER <-> SLAVE sync: Finished with success
```


master端创建key测试复制
```
127.0.0.1:6379> set name da
OK
```

slave查看是否同步过来key
```
127.0.0.1:6380> keys *
1) "name"
2) "age"
127.0.0.1:6380> get name
"da"
```


## 复制状态
通过`info replication`复制状态信息，master端和slave数据不一样

master端复制状态信息，可以看到当前角色，slave地址信息，状态、偏移量等
```
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=1077,lag=0
master_repl_offset:1077
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:1076
```

slave端复制状态信息，可以看到master的地址信息、状态，以及slave的信息，例如只读slave，偏移量等
```
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:1371
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

## 断开复制
slave端断开复制，断开后不会删除数据，只是不同步原主的新数据
```
127.0.0.1:6380> slaveof no one
OK
```


master测试修改数据
```
127.0.0.1:6379> set name da
OK
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:2791
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:2790
```

slave检查数据
```
127.0.0.1:6380> get name 
"yo"
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:2550
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```


## 安全
masterauth master_auth_password

## 延迟
 no
redis通过repl-disable-tcp-nodelay来控制tcp_nodelay，默认为no

意味尽可能的减少延迟，这是用于同机柜或者同机房等低网络延迟(代价)的环境

如果跨机房同步的话就需要考虑到网络传输的代价，所以在网络延迟较高是开启nodelay能够有效的减少网络的消耗，这点有点类似于pipeline
