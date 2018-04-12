# RDB

RDB持久化是指Redis进程把数据生成快照保存到文件系统的过程，触发RDB持久化的方式分为两种，手动触发和自动触发


## 手动触发
### 如何手动触发RDB持久化？ 手动持久化的有什么问题？

**手动触发持久化命令**
* save: 阻塞当前Redis服务，直到save持久化完成，不建议在访问量较大的生产中使用
* bgsave: fork创建子进程执行save持久化，阻塞只发生fork阶段，通常时间非常短，通过日志可以看到redis默认内部使用的bgsave

save日志
```
1983:M 03 Jan 08:56:32.199 * DB saved on disk
```

bgsave日志
```
1983:M 03 Jan 08:59:08.960 * Background saving started by pid 13078
13078:C 03 Jan 08:59:08.971 * DB saved on disk
13078:C 03 Jan 08:59:08.991 * RDB: 0 MB of memory used by copy-on-write
1983:M 03 Jan 08:59:09.006 * Background saving terminated with success
```


## 自动触发
**1. save m n相关配置，例如如果m秒内有n次修改的话，则自动触发RDB持久化执行bgsave**
  例如默认redis.conf配置
  ```
  # 900秒内有1次修改则触发
  save 900 1
  # 300秒内有10次修改则触发
  save 300 10
  # 60秒内有10000次修改则触发
  save 60 10000
  ```

  通过日志也可以看到对应触发的情况
  ```log
  1983:M 02 Jan 09:49:41.052 * 10 changes in 300 seconds. Saving...
  1983:M 02 Jan 09:49:41.052 * Background saving started by pid 30144
  30144:C 02 Jan 09:49:41.060 * DB saved on disk
  30144:C 02 Jan 09:49:41.061 * RDB: 0 MB of memory used by copy-on-write
  1983:M 02 Jan 09:49:41.153 * Background saving terminated with success
  1983:M 02 Jan 09:57:45.945 * DB saved on disk
  1983:M 02 Jan 10:12:46.034 * 1 changes in 900 seconds. Saving...
  1983:M 02 Jan 10:12:46.034 * Background saving started by pid 30546
  30546:C 02 Jan 10:12:46.043 * DB saved on disk
  30546:C 02 Jan 10:12:46.043 * RDB: 0 MB of memory used by copy-on-write
  1983:M 02 Jan 10:12:46.134 * Background saving terminated with success
  ```
**~~2. slave节点执行权利复制操作，master节点则自动触发RDB持久化执行bgsave并将数据文件发送给slave节点以同步数据(未测试)~~**

**3. 执行debug reload命令时则会触发RDB持久化执行save**
  执行命令
  ```
  127.0.0.1:6379> debug reload
  OK
  ```
  日志输出
  ```
  root@iZ947mgy3c5Z:/opt/redis# tail -0f data/redis.log 
  1983:M 03 Jan 09:12:38.905 * DB saved on disk
  1983:M 03 Jan 09:12:38.905 # DB reloaded by DEBUG RELOAD
  ```
  
**4. 默认情况如果执行shutdown时如果没有开启AOF持久化功能，也会触发RDB持久化执行bgsave**
  命令执行
  ```
  root@iZ947mgy3c5Z:/opt/redis# redis-cli shutdown
  ```
  日志输出
  ```
  1983:M 03 Jan 09:14:24.998 # User requested shutdown...
  1983:M 03 Jan 09:14:24.998 * Saving the final RDB snapshot before exiting.
  1983:M 03 Jan 09:14:25.011 * DB saved on disk
  1983:M 03 Jan 09:14:25.011 * Removing the pid file.
  1983:M 03 Jan 09:14:25.012 # Redis is now ready to exit, bye bye...
  ```
  
## 持久化流程
简单的草图
![](/assets/rdb_persistence.jpg)

步骤说明
1. 手动/触发RDB持久化执行bgsave
2. redis父进程得知到需要执行RDB持久化
3. redis父进程检查是否存在子进程正在执行RDB持久化
4. 如果有子进程正在执行持久化直接返回
4. 没有子进程的话fork子进程，fork操作期间，父进程会阻塞，可以通过info stats中的latest_fork_usec查看到最近的fork阻塞时间，单位是微秒，子进程fork创建完毕后，子进程向父进程返回`Background saving started`，父进程收到信息后不再阻塞，父进程更新info stats中的latest_fork_usec，下面是几个相关参数
5. 父进程接收新的命令
6. 子进程执行RDB持久化，首先创建rdb数据文件，根据父进程的内存数据生成快照，然后对原有数据文件进行原子替换，根据持久化执行情况更改info中的信息
 * rdb_changes_since_last_save: 上次持久化后新的修改数
 * rdb_bgsave_in_progress: 标识是否有子进程正在执行持久化
 * rdb_last_save_time: 上次RDB持久化成功的时间戳
 * rdb_last_bgsave_status: 上次RDB持久化的状态
 * rdb_last_bgsave_time_sec: 上次RDB持久化花费的时间(秒)
 * rdb_current_bgsave_time_sec: 如果有持久化子进程，子进程当前正在执行的RDB持久化的秒级时间


## RDB数据文件

RDB持久化出来的文件，存在dir配置下以dbfilename配置命名，例如
```
127.0.0.1:6379> config get dir
1) "dir"
2) "/opt/redis-3.0.7/data"
127.0.0.1:6379> config get dbfilename
1) "dbfilename"
2) "dump.rdb"
127.0.0.1:6379> 
root@iZ947mgy3c5Z:/opt/redis# ls -al /opt/redis-3.0.7/data/dump.rdb
-rw-r--r-- 1 root root 42 Jan  3 09:59 /opt/redis-3.0.7/data/dump.rdb
``` 

**dir和dbfilename支持动态修改，这也就意味着，如果磁盘出现问题，可以通过动态调整数据文件路径避免持久化失败或者重启redis配置的问题**

实验效果
```
root@iZ947mgy3c5Z:/opt/redis# mkdir /data/redis_6379/data -p
root@iZ947mgy3c5Z:/opt/redis# redis-cli
127.0.0.1:6379> config set dir "/data/redis_6379/data"
OK
127.0.0.1:6379> config set dbfilename "redis_6379.rdb"
OK
127.0.0.1:6379> config get dir
1) "dir"
2) "/data/redis_6379/data"
127.0.0.1:6379> config get dbfilename
1) "dbfilename"
2) "redis_6379.rdb"
127.0.0.1:6379> bgsave
Background saving started
127.0.0.1:6379> 
root@iZ947mgy3c5Z:/opt/redis# ls -al /data/redis_6379/data/redis_6379.rdb
-rw-r--r-- 1 root root 62 Jan  3 10:13 /data/redis_6379/data/redis_6379.rdb
```

**检查数据文件的有效性**
```
root@iZ947mgy3c5Z:/opt/redis# redis-check-dump /data/redis_6379/data/redis_6379.rdb 
==== Processed 6 valid opcodes (in 45 bytes) ===================================
CRC64 checksum is OK
```

## RDB持久化总结

### 优点
1. 数据文件体积小，非常适合定期备份，全量复制，例如每6个小时备份一次，然后将数据文件同步到远程主机，用作灾难恢复
2. 加载速度相较于aof快很多

### 缺点
1. 无法实现实时备份/秒级备份，而且fork在访问量较高的场景属于成本比较高的操作
2. rdb数据文件在redis新老版本之间存在不兼容的问题

## RDB持久化深入
http://blog.dujiong.net/2016/11/27/Redis-RDB/



## 补充说明

**原子替换**
RDB文件一旦被创建，就不会进行任何修改。当服务器要创建一个新的RDB文件时，它先将文件的内容保存在一个临时文件里，当临时文件写入完毕时，程序才原子地使用临时文件替换原来的RDB文件。即，无论何时，复制RDB文件都是绝对安全的。