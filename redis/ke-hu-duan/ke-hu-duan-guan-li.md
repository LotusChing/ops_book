# 客户端管理

类似MySQL中的`show processlist`，但是信息要比它多非常多

### client list: 列出客户端连接信息
```
127.0.0.1:6379> client list
id=1905 addr=127.0.0.1:16099 fd=5 name= age=2 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```

#### 基础信息
* id: 连接/会话标识
* addr: 远程客户端IP地址和端口
* fd: 文件描述符
* db: 操作的数据库
* sub: 已订阅频道的数量
* ~~psub: 已订阅模式的数量(不太清楚什么意思？)~~
* multi: 在事务中被执行的命令数量


#### 输入缓冲信息
* qbuf: 查询缓冲区的大小(字节为单位， 0 表示没有分配查询缓冲区)
* qbuf-free: 查询缓冲区剩余大小(Redis硬编码一个连接最大1G，超过1G后Redis会自动关闭客户端连接)

**缓冲使用不当的情况**
1. 客户端查询缓冲超过1G会被客户端关闭连接
2. 查询缓冲不受maxmemory控制，当Redis在内存中的数据和查询缓冲数据相加大于maxmemory，有可能导致数据丢失、键值淘汰、OOM等异常情况

**什么情况下会出现查询缓冲过大？**
1. Redis的处理速度跟不上查询缓冲区的输入速度
2. Redis发生了阻塞，导致查询缓冲过大

**如何监控查询缓冲使用情况？**
1. 通过定期执行`client list`获取`qbuf`和`qbuf-free`，从而分析是那个客户端连接的问题
2. 通过定期执行`info clients`获取`client_biggest_input_buf`，基于该值做阈值判断，如果超过10M则报警通知

|方式|优点|缺点|
|:-:|:--:|:--:|
|client list|能够详细的看到每隔客户端的连接信息，便于分析排查问题|当客户端连接过大时，执行client list可能会导致Redis阻塞|
|info clients|执行速度比client list快|数据较为简单，不利于排查问题|


#### 输出缓冲信息
* obl: 固定缓冲区的长度（字节为单位，0 表示没有分配输出缓冲区）
* oll: 动态缓冲列表的长度（当缓冲区已满时，回复在此列表中排队）
* omem: 输出缓冲区和输出列表占用的内存总量(字节)


Redis为每个客户端连接提供了输出缓冲区，它的作用就是保存命令执行结果并返回给客户端，输出缓冲区和查询缓冲区不同的是，输出缓冲区分为三种
1. 普通客户端缓冲区
2. slave客户端缓冲区
3. pub/sub发布订阅客户端缓冲区

三种客户端都可以通过`client-output-buffer-limit`进行配置，而且针对客户端的不同配置不同的限制

`client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>`
* class: 客户端的类型，normal、slave、pubsub
* hard limit: 硬限制，如果输出缓冲区超过该限制，则立即关闭客户端连接
* soft limit: 软限制，如果输出缓冲区超过该限制，并且持续超过soft seconds时长，则关闭客户端连接

Redis默认配置
client-output-buffer-limit normal 0 0 0 
client-output-buffer-limit slave 256mb 64m 60
client-output-buffer-limit pubsub 32mb 8mb 60

> Tips: 输出缓冲其实固定缓冲区(字节数组16K)和动态缓冲区(列表)，固定缓冲区用于存放较小的命令执行结果，动态缓冲区列表则存放较大的命令执行结果，例如mget、hgetall、smembers等，如果固定缓冲区用满后，就会把数据添加进动态缓冲区列表中

**如何监控输出缓冲区？**
1. client list
2. info clients

**配置输出缓冲区的注意事项有哪些？**
1. normal配置hard|soft limit，避免大量的输出缓冲导致数据丢失、键值淘汰、OOM
2. 当master写入量大时通常slave的输出缓冲区会比较大，所以建议适当增大slave的限制，避免slave客户端连接被KILL导致重新复制
3. 限制让缓冲区增大的命令，例如monitor


#### 监控客户端连接状态

* age: 以秒计算的已连接时长
* idle: 以秒计算空闲时长

如果age=idle说明连接一直处于空闲，py脚本通过sleep模拟，为了避免redis被大量空闲连接导致新连接无法创建，redis提供了下面两个参数
* maxclients：redis最大的客户端连接数，默认10000
* timeout: 当客户端连接idle超过timeout时，则客户端被关闭，默认永不超时

**测试未配置timeout**
窗口1：执行测试脚本
    ```python
    import time
    import redis
    client = redis.StrictRedis(host="127.0.0.1", port=6379)
    
    print(client.get("hello"))
    time.sleep(30)
    print(client.get("hello"))
    ```
窗口二：间隔执行client list查看
```
id=1909 addr=127.0.0.1:5356 fd=5 name= age=1 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
...
id=1909 addr=127.0.0.1:5356 fd=5 name= age=30 idle=30 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
```

**测试配置timeout为10秒**
配置超时
```
127.0.0.1:6379> config set timeout 10
OK
```

窗口一：执行py脚本

窗口二：查看客户端连接状态
```
...
127.0.0.1:6379> client list
id=1907 addr=127.0.0.1:58183 fd=6 name= age=1574 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
id=1910 addr=127.0.0.1:5775 fd=5 name= age=10 idle=10 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get

127.0.0.1:6379> client list
id=1907 addr=127.0.0.1:58183 fd=6 name= age=1575 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```

#### 客户端类型
* flags: 客户端flag

|序号|flag|描述|
|1|N|普通客户端|
|2|M|当前客户端是Master节点|
|3|S|当前客户端是Slave节点|
|4|O|当前客户端正在执行monitor|
|5|x|正在执行事务|
|6|b|正在等待阻塞事件|
|7|d|一个受watch监视的key被修改，EXEC将失败|
|8|u|客户端未被阻塞|
|9|U|客户端使用sock套接字连接|
|10|r|客户端在集群中处于只读模式|
|11|c|客户端返回完整结果后关闭连接|
|12|A|客户端尽快的关闭|


#### 命令及类型
* events: 读还是写，还是读写
* cmd: 客户端连接最后一次执行的命令，不包括参数


#### 客户端名称
* name: 客户端端的名称，`clinet setName client getName`
```
127.0.0.1:6379> client setName redis-120
OK
127.0.0.1:6379> client getName
"redis-120"
127.0.0.1:6379> client list
id=1914 addr=127.0.0.1:14206 fd=5 name=redis-120 age=24 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```


### client kill: 手动关闭客户端连接
手动关闭执行客户端连接
```
127.0.0.1:6379> client list
id=1918 addr=127.0.0.1:15224 fd=5 name= age=7 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
id=1919 addr=127.0.0.1:15240 fd=6 name= age=5 idle=5 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
127.0.0.1:6379> CLIENT kill 127.0.0.1:15240
OK
127.0.0.1:6379> client list
id=1918 addr=127.0.0.1:15224 fd=5 name= age=15 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```


## 客户端连接配置
* timeout
* maxclients
* tcp-keepalive: 检测并释放超过60秒的tcp死连接，默认为0不检测
* tcp-backlog: 三次握手后，会将接收的连接放入队列中，tcp-backlog就是队列大小，Redis默认是511，不过这个值受linux操作系统的/proc/sys/core/somaxconn限制，如果系统小于511，redis日志中会提示warning，`1983:M 27 Dec 13:15:36.762 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.`


## 监控客户端连接指标
info clients
* connected_clients: 当前客户端连接数:
* client_longest_output_list: 当前所有输出缓冲区队列对象个数的最大值
* client_biggest_input_buf: 当前所有查询缓冲区中占用内存的最大值
* blocked_clients: 被阻塞的客户端连接

info stats
* total_connections_received: 从启动以来建立了多少客户端连接
* rejected_connections: 从启动以来拒绝了多少个客户端连接


## 客户端常见异常

## 客户端磁场案例分析