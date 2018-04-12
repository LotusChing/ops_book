# Slow Query Log

## 相关参数
|Variable|Description|Remark|
|:------:|:---------:|:-----:|
|`slow_query_log`|慢查询日志开关||
|`slow_query_log_file`|慢查询日志文件位置|日志文件不要和数据文件放到一起|
|`long_query_time`|慢查询阈值|`5.5`以后支持毫秒|
|`min_examined_row_limit`|扫描记录低于指定行的不记录||
|`log_queries_not_using_indexes`|记录没走索引的`sql`||
|`log_throttle_queries_not_using_indexes`|限制每分钟记录没走索引的`sql`次数|5.6以后支持，用来避免因为大量重复慢查询`sql`导致`IO`消耗在写日志|
|`log_slow_admin_statements`|记录管理操作，例如`ALTER/ANALYZE TABLE`||
|`log_output`|慢查询日志的格式，[`FILE`、`TABLE`、`NONE`]||
|`log_slow_slave_statements`|再`slave`上开启满查询日志||
|`log_timestamps`|写入时间戳|`5.7`以后支持|


通过参数可以总结出来记录慢查询SQL的规则大概像是这样
![](/assets/Q1`KOHS0][8LH`{I(5P}UEH.png)

## 慢查询日志文件

```
$ more slow.log

# Time: 2017-06-26T19:32:00.563082Z
# User@Host: root[root] @ localhost []  Id: 50097
# Query_time: 3.000817  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1498505520;
select sleep(3);
/usr/local/mysql/bin/mysqld, Version: 5.7.18 (MySQL Community Server (GPL)). started with:
Tcp port: 3306  Unix socket: /tmp/mysql.sock
Time                 Id Command    Argument
# Time: 2017-06-26T20:42:00.666767Z
# User@Host: root[root] @ localhost []  Id: 50133
# Query_time: 0.002559  Lock_time: 0.002345 Rows_sent: 1  Rows_examined: 1
SET timestamp=1498509720;
select * from da.t2;
```
MySQL提供了一个分析慢查询日志的工具，mysqldumpslow
```
Reading mysql slow query log from Da-slow.log
Count: 2  Time=3.03s (6s)  Lock=0.00s (0s)  Rows=1.0 (2), root[root]@localhost
  select sleep(N)

Count: 1  Time=0.10s (0s)  Lock=0.00s (0s)  Rows=1.0 (1), root[root]@localhost
  select * from t2

Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=1.0 (1), root[root]@localhost
  select * from da.t2
```

`mysqldumpslow`支持一些好玩的参数，例如根据`count`倒序排序然后取出`top1`
```
$ mysqldumpslow -s c Da-slow.log -t 1

Reading mysql slow query log from Da-slow.log
Count: 3  Time=3.02s (9s)  Lock=0.00s (0s)  Rows=1.0 (3), root[root]@localhost
  select sleep(N)
```

## 慢查询日志表
由于默认`log_output`参数是`FILE`，所以需要手动修改下
```
mysql> set global log_output='TABLE';
mysql> select sleep(3);
mysql> select * from mysql.slow_log\G
*************************** 1. row ***************************
    start_time: 2017-06-27 04:27:24.285105
     user_host: root[root] @ localhost []
    query_time: 00:00:03.001014
     lock_time: 00:00:00.000000
     rows_sent: 1
 rows_examined: 0
            db: mysql
last_insert_id: 0
     insert_id: 0
     server_id: 11
      sql_text: select sleep(3)
     thread_id: 50122
1 row in set (0.00 sec)
```
不过`slow_log`表示`csv`存储引擎，所以性能会比较低，我们可以修改为`myisam`，不过需要先临时把慢日志关闭下
```
mysql> set slow_query_log=0;
mysql> alter table slow_log engine=myisam;
Query OK, 1 row affected (0.17 sec)
Records: 1  Duplicates: 0  Warnings: 0
```


## 总结
简单总结下`FILE`和`TABLE`的优缺点
* FILE
 * 缺点：查询不如表灵活，每天切割日志
 * 优点：自定义文件位置，灵活且避免影响数据盘IO
* TABLE
 * 缺点：备份时容易备份进去，一定程度上影响数据盘IO
 * 优点：查询条件灵活方便