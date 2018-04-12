# 慢查询

Redis也支持像MySQL一样支持将查询时间超过阈值命令记录到满查询日志中

Redis命令执行分为简单4个步骤
1. 接收命令
2. 命令排队
3. 执行命令
4. 返回结果

Redis是在命令执行前开始计时，执行结束后停止计时

## 配置参数
* slowlog-log-slower-than: 阈值，单位是微秒，默认是10000 = 10ms
* slowlog-max-len: 总记录数，Redis会使用列表来存放慢查询，超过最大条数则删除最早添加的第一条


## 配置方法
* 配置文件 
* config

**通过config配置慢查询**
```
127.0.0.1:6379> config set slowlog-log-slower-than 10001
OK
127.0.0.1:6379> config set slowlog-max-len 100
OK
127.0.0.1:6379> config rewrite
OK
```


**获取所有慢查询信息**
```
127.0.0.1:6379> SLOWLOG GET
1) 1) (integer) 2
   2) (integer) 1514422825
   3) (integer) 244062
   4) 1) "config"
      2) "rewrite"
2) 1) (integer) 1
   2) (integer) 1514359832
   3) (integer) 25502
   4) 1) "DEL"
      2) "user_name1"
3) 1) (integer) 0
   2) (integer) 1514352416
   3) (integer) 10312
   4) 1) "FLUSHALL"
```

每条慢查询有4个属性，慢查询ID、发生时间戳、命令耗时μs、命令及参数

**获取指定条数的慢查询信息**
```
127.0.0.1:6379> SLOWLOG GET 2
1) 1) (integer) 2
   2) (integer) 1514422825
   3) (integer) 244062
   4) 1) "config"
      2) "rewrite"
2) 1) (integer) 1
   2) (integer) 1514359832
   3) (integer) 25502
   4) 1) "DEL"
      2) "user_name1"
```

**统计慢查询数量**
```
127.0.0.1:6379> SLOWLOG LEN
(integer) 3
```

**清理慢查询信息**
```
127.0.0.1:6379> SLOWLOG LEN
(integer) 3
127.0.0.1:6379> SLOWLOG RESET
OK
127.0.0.1:6379> SLOWLOG LEN
(integer) 0
```

## 最佳实践
1. slowlog-log-slower-than: 对于高流量的情况，减小阈值为1毫秒，由于Redis是单线程，如果一个命令耗时1毫秒，那么意味着Redis所支撑的OPS最多不会超过1000，所以需要记录，但是，这样会不会导致大量慢查询被记录从而更消耗性能呢？
2. slowlog-max-len: 建议调高慢查询列表，Redis会对长命令截断，所以不会占用大量内存，建议配置1000以上
3. 当客户端出现超时，应及时检查Redis是否存在级联阻塞
4. 可以酌情考虑将慢查询持久化到MySQL，CacheCloud

