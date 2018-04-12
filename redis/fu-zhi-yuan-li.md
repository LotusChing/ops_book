# 复制原理


## 复制特点
* 一个master可以有多个slave
* 一个slave只能有一个master
* 数据流向是单项的，master到slave

## run id、偏移量
* run id: redis实例的标识，每次重启后会重新生成run id，当master端run id发生变化，slave会重新全量复制一遍master
* 偏移量: 主要用来描述主从之间数据情况，如果master、slave不同说明可能存在不一致的请，通过info replication
  ```
  slave0:ip=127.0.0.1,port=6380,state=online,offset=339588,lag=0
master_repl_offset:339588
  ```

## 简单原理

草图
![](/assets/master_slave_replication_simple_flow.jpg)](/assets/master_slave_replication_simple_flow.jpg)


简单描述

1. slave保存master的地址信息，例如IP端口
2. slave发起连接master ip:port
3. 如果连接成功slave则发送ping命令检测，master端能否正常处理命令
4. 如果master端能够返回pong的话，说明master端当前正常
5. 检测master是否开启密码验证，如果开启了密码验证，slave通过masterauth密码进行验证
6. 验证成功则开始请求同步master数据，master调用bgsave生成rdb数据文件，然后把数据发送给slave
7. slave清空本地老数据，载入新的数据到内存中
8. slave持续接收master发送的写入数据


## 补充说明
全量复制
![](/assets/redis-master-slave-replication-full-detail.jpg)

部分复制
![](/assets/redis-master-slave-replication-partial.jpg)

## 全量复制开销
1. master：bgsave时间
2. master：rdb传输时间
3. slave：清空数据时间
4. slave：载入rdb时间
5. slave：aof开启时可能会发生重写aof


