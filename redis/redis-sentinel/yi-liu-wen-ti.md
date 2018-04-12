# 遗留问题

## 主从分离时从节点下线客户端无法感知到

首先我错了，我没有验证课程中讲师的说法

后来，我准备了一个不断读取数据的Py脚本，然后通过`kill -9 <pid>`实验模拟一个Slave宕机，看是否会读取数据的脚本是否会因为连接不上原来的slave而报错



## 查看客户端读取那台slave
### slave1的默认情况
这是Slave1默认的client list，没有看到客户端ip及get命令，说明py脚本连接的不是这台
```python
root@consul-app:~# redis-cli -h 192.168.2.20
192.168.2.20:6379> CLIENT LIST
id=4 addr=192.168.2.20:57548 fd=9 name=sentinel-d3c1af10-cmd age=460 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
id=5 addr=192.168.2.20:57549 fd=10 name=sentinel-d3c1af10-pubsub age=460 idle=1 flags=N db=0 sub=1 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=subscribe
id=6 addr=192.168.2.50:33227 fd=12 name=sentinel-2986e7bf-cmd age=460 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=publish
id=7 addr=192.168.2.50:33228 fd=13 name=sentinel-2986e7bf-pubsub age=460 idle=1 flags=N db=0 sub=1 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=subscribe
id=2 addr=192.168.2.30:52908 fd=6 name=sentinel-b48e3696-cmd age=460 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
id=8 addr=192.168.2.30:6379 fd=7 name= age=460 idle=1 flags=M db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=publish
id=3 addr=192.168.2.30:52909 fd=8 name=sentinel-b48e3696-pubsub age=460 idle=1 flags=N db=0 sub=1 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=subscribe
id=10 addr=192.168.2.50:33416 fd=5 name= age=2 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
192.168.2.20:6379> 
```

### slave2的默认情况
可以看到一条ip为2.99的客户端IP，以及对应的Get命令，说明py脚本连接到这的slave2了
```python
root@consul-app:~# redis-cli -h 192.168.2.50
192.168.2.50:6379> CLIENT LIST
id=7 addr=192.168.2.20:47037 fd=13 name=sentinel-d3c1af10-pubsub age=455 idle=1 flags=N db=0 sub=1 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=subscribe
id=14 addr=192.168.2.50:33076 fd=9 name= age=2 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
id=8 addr=192.168.2.30:6379 fd=8 name= age=455 idle=1 flags=M db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=publish
id=12 addr=192.168.2.99:64708 fd=5 name= age=76 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
id=2 addr=192.168.2.50:60855 fd=6 name=sentinel-2986e7bf-cmd age=455 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
id=3 addr=192.168.2.50:60856 fd=7 name=sentinel-2986e7bf-pubsub age=455 idle=1 flags=N db=0 sub=1 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=subscribe
id=4 addr=192.168.2.30:56198 fd=10 name=sentinel-b48e3696-cmd age=455 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
id=5 addr=192.168.2.30:56199 fd=11 name=sentinel-b48e3696-pubsub age=455 idle=1 flags=N db=0 sub=1 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=subscribe
id=6 addr=192.168.2.20:47036 fd=12 name=sentinel-d3c1af10-cmd age=455 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
```

## 测试模拟宕机
这里我获取到redis的pid，然后kill掉它
```
root@consul-app:~# ps uax|grep redis
root      5898  0.8  0.4  36844  2276 ?        Ssl  13:34   0:10 redis-sentinel *:26379 [sentinel]                       
root      5989  0.5  0.4  36844  2232 ?        Ssl  13:46   0:02 redis-server 192.168.2.50:6379                           
root      6020  0.0  0.1  10476   936 pts/5    S+   13:54   0:00 grep --color=auto redis
root@consul-app:~# kill -9 5989
```



## 检查脚本输出和slave客户端连接
脚本没有任何报错，在slave1上看到了2.99的客户端连接，惊呆了，这个redis-setinel的客户端做的真的没毛病，棒棒棒棒棒棒

```python
root@consul-app:~# redis-cli -h 192.168.2.20
192.168.2.20:6379> CLIENT LIST
id=12 addr=192.168.2.50:33653 fd=11 name= age=1 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
id=11 addr=192.168.2.99:64794 fd=5 name= age=9 idle=6 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
id=4 addr=192.168.2.20:57548 fd=9 name=sentinel-d3c1af10-cmd age=486 idle=1 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
id=5 addr=192.168.2.20:57549 fd=10 name=sentinel-d3c1af10-pubsub age=486 idle=0 flags=N db=0 sub=1 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=subscribe
id=6 addr=192.168.2.50:33227 fd=12 name=sentinel-2986e7bf-cmd age=486 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=publish
id=7 addr=192.168.2.50:33228 fd=13 name=sentinel-2986e7bf-pubsub age=486 idle=0 flags=N db=0 sub=1 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=subscribe
id=2 addr=192.168.2.30:52908 fd=6 name=sentinel-b48e3696-cmd age=486 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=ping
id=8 addr=192.168.2.30:6379 fd=7 name= age=486 idle=0 flags=M db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=publish
id=3 addr=192.168.2.30:52909 fd=8 name=sentinel-b48e3696-pubsub age=486 idle=0 flags=N db=0 sub=1 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=subscribe
192.168.2.20:6379> 
```