# Redis Shell

* redis-cli
* redis-server
* redis-benchmark

## redis-cli

**重复执行命令: -r**
    ```
    root@iZ947mgy3c5Z:~# redis-cli -r 3 ping
    PONG
    PONG
    PONG
    
    
    root@iZ947mgy3c5Z:~# redis-cli -r 3 info|grep "used_memory_human"
    used_memory_human:796.20K
    used_memory_human:796.20K
    used_memory_human:796.20K
    ```

**间隔重复执行命令: -r -i **
    ```
    root@iZ947mgy3c5Z:~# redis-cli -r 2 -i 1 ping
    PONG
    PONG
    root@iZ947mgy3c5Z:~# redis-cli -r 2 -i 1 info|grep "used_memory_human"
    used_memory_human:796.20K
    used_memory_human:796.20K
    ```

**通过标准输入作为redis-cli最后一个参数(没懂啥用): -x**
    ```
    root@iZ947mgy3c5Z:~# echo "Da" | redis-cli -x set name 
    OK
    
    root@iZ947mgy3c5Z:~# echo "ping" | redis-cli 
    PONG
    
    root@iZ947mgy3c5Z:~# redis-cli get name
    "Da\n"
    ```

**连接Redis Cluster(待补充): -c**

**通过模拟slave获取Redis执行的命令: --slave**

窗口一：
    ```
    root@iZ947mgy3c5Z:~# redis-cli get name
    "Da\n"
    root@iZ947mgy3c5Z:~# redis-cli --slave
    SYNC with master, discarding 61 bytes of bulk transfer...
    SYNC done. Logging commands from master.
    "PING"
    "PING"
    "PING"
    "SELECT","0"
    "set","name","Da"
    "PING"
    ```

窗口二：
    ```
    root@iZ947mgy3c5Z:~# redis-cli 
    127.0.0.1:6379> set name "Da"
    OK
    127.0.0.1:6379> 
    ```

**持久化数据成文件: --rdb(待补充)**

**多条命令封装成Redis通讯协议支持的数据格式已完成批量执行: --pipe(待补充)**

**扫描Redis中的大键，这些bigkey有可能就是性能瓶颈： --bigkeys**

**执行外部lua脚本: --lua(待补充)**

**测试客户端到Redis的延迟: --latency --latency-history --latency-dist**

`--latency`会持续测试并实时计算延迟
```
root@iZ947mgy3c5Z:~# redis-cli --latency -h 127.0.0.1
min: 0, max: 1, avg: 0.09 (4886 samples)^C
```

`--latency-history`也会持续测试并实时计算延迟，但每隔15秒新建一行
```
root@iZ947mgy3c5Z:~# redis-cli --latency-history -h 127.0.0.1
min: 0, max: 1, avg: 0.08 (1467 samples) -- 15.01 seconds range
min: 0, max: 1, avg: 0.07 (1467 samples) -- 15.01 seconds range
min: 0, max: 1, avg: 0.07 (1464 samples) -- 15.00 seconds range
```

至于`--latency-dist`讲道理，当看到它执行效果时我一个没忍住笑了出来
![](/assets/redis-cli-latency-dist.jpg)


**实时获取Redis当前状态信息: --stat**
```
root@iZ947mgy3c5Z:~# redis-cli --stat
------- data ------ --------------------- load -------------------- - child -
keys       mem      clients blocked requests            connections          
3          1.80M    2       0       11325 (+0)          26          
3          1.80M    2       0       11326 (+1)          26          
3          1.80M    2       0       11327 (+1)          26          
3          1.80M    2       0       11328 (+1)          26          
3          1.80M    2       0       11329 (+1)          26          
3          1.80M    2       0       11330 (+1)          26          
3          1.80M    2       0       11331 (+1)          26          
```

**终端获取原始格式(例如中文)值(default when STDOUT is not a tty): --raw**

官方说明
    ```
    --raw              Use raw formatting for replies (default when STDOUT is
                     not a tty).
    --no-raw           Force formatted output even when STDOUT is not a tty.
    ```

    ```
    root@iZ947mgy3c5Z:~# redis-cli get name
    "\xe6\x95\x9b\xe9\x9d\x92"
    root@iZ947mgy3c5Z:~# redis-cli --raw get name
    敛青
    ```


## redis-server
**测试Redis能否正常使用一定量的内存: --test-memory**
```
root@iZ947mgy3c5Z:~# redis-server --test-memory 10M
....
Your memory passed this test.
Please if you are still in doubt use the following two tools:
1) memtest86: http://www.memtest86.com/
2) memtester: http://pyropus.ca/software/memtester/
```
Your memory passed this test. 表示正常，同时Redis推荐了更专业的两款内存测试工具



## redis-benchmark

`-c`: 并发数
`-n`: 请求数
`-h`: Redis主机
`-q`: 简练输出
`-r`: 压测使用多建，键名后缀使用随机数，10000代表后四位随机
`-t`: 单项功能基准测试
`-k`: 是否启用keepalive，1使用，0不用
`--csv`: 测试结果csv格式
```
[root@bj-vmware-test1 redis]# redis-benchmark -c 100 -n 20000 -h 192.168.2.20 -q -r 10000
PING_INLINE: 10735.37 requests per second
PING_BULK: 12953.37 requests per second
SET: 12143.29 requests per second
GET: 8421.05 requests per second
INCR: 9136.59 requests per second
LPUSH: 10917.03 requests per second
LPOP: 9955.20 requests per second
SADD: 10449.32 requests per second
SPOP: 11918.95 requests per second
LPUSH (needed to benchmark LRANGE): 9857.07 requests per second
LRANGE_100 (first 100 elements): 7176.18 requests per second
LRANGE_300 (first 300 elements): 2977.96 requests per second
LRANGE_500 (first 450 elements): 2208.48 requests per second
LRANGE_600 (first 600 elements): 1738.07 requests per second
MSET (10 keys): 9074.41 requests per second
``` 

单项基准测试
```
root@iZ947mgy3c5Z:~# redis-benchmark -c 10 -n 100 -h 127.0.0.1 -t get,set -q
SET: 33333.33 requests per second
GET: 50000.00 requests per second
```