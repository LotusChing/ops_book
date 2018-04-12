# 表的创建和维护

## 临时表

临时表顾名思义是临时的，生命周期仅限于当前会话，当前会话结束，临时表和表中的数据就会被释放掉

### 如何创建临时表
```sql
mysql> CREATE TEMPLORARY TABLE a(id int);
```

### 临时表用的是什么存储引擎？
为什么这里用的是InnoDB？
```
mysql> show create table a\G
*************************** 1. row ***************************
       Table: a
Create Table: CREATE TEMPORARY TABLE `a` (
  `a` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)
```

原因在这！ 因为这个参数配置了默认临时存储引擎，所以创建临时表时会根据`default_tmp_storage_engine`参数中配置的engine来创建
```sql
mysql> show global variables like '%default_tmp_%';
+----------------------------+--------+
| Variable_name              | Value  |
+----------------------------+--------+
| default_tmp_storage_engine | InnoDB |
+----------------------------+--------+
1 row in set (0.00 sec)
```


### 临时表在哪？内存中？文件系统中？
一直以来我都是认为临时表使用在排序中的，而且临时表实在内存中的，但是今天通过学习后才发现，不是这样的，Look！

MySQL有个参数叫做`tmpdir`，先到这里看下
```sql
mysql> show global variables like 'tmpdir';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| tmpdir        | /tmp  |
+---------------+-------+
1 row in set (0.00 sec)
```

难道这个`#sql2993_4_0.frm`是a的表定义文件？
```
[root@Da dbfile1]# ls /tmp/
functions  letsencrypt-auto  minion.pem  #sql2993_4_0.frm
```

再创建一个临时表试下，果然又出来一个，看来临时表确实没有存在内存中，呃，不过这只是表定义文件，那表中的数据呢？
```sql
mysql> CREATE TEMPLORARY TABLE b(id int);
mysql> system ls /tmp
functions  letsencrypt-auto  minion.pem  #sql2993_4_0.frm  #sql2993_4_1.frm
```

因为表使用的InnoDB存储引擎，看下能够从InnoDB的参数中找到线索呢？ 嗯？ 看到InnoDB有个`innodb_temp_data_file_path`参数，会不会跟这个参数有关系呢？
```sql
mysql> show global variables like 'innodb_temp_data%';
+----------------------------+-----------------------+
| Variable_name              | Value                 |
+----------------------------+-----------------------+
| innodb_temp_data_file_path | ibtmp1:12M:autoextend |
+----------------------------+-----------------------+
```

在`datadir`下找到了`ibtmp1`这个文件
```sql
mysql> system ls -ahl /data/dbfile1/ib*
-rw-r-----. 1 mysql mysql 374 Aug  5 07:39 /data/dbfile1/ib_buffer_pool
-rw-r-----. 1 mysql mysql 12M Aug  5 08:01 /data/dbfile1/ibdata1
-rw-r-----. 1 mysql mysql 48M Aug  5 07:51 /data/dbfile1/ib_logfile0
-rw-r-----. 1 mysql mysql 48M Jul  3 04:16 /data/dbfile1/ib_logfile1
-rw-r-----. 1 mysql mysql 12M Aug  5 08:09 /data/dbfile1/ibtmp1
```

到底是不是存储在这个文件呢？ 我们插入一条数据试下，看文件修改时间是否改变
```sql
mysql> insert into a select 1;
mysql> system ls -ahl /data/dbfile1/ib*
-rw-r-----. 1 mysql mysql 374 Aug  5 07:39 /data/dbfile1/ib_buffer_pool
-rw-r-----. 1 mysql mysql 12M Aug  5 08:01 /data/dbfile1/ibdata1
-rw-r-----. 1 mysql mysql 48M Aug  5 07:51 /data/dbfile1/ib_logfile0
-rw-r-----. 1 mysql mysql 48M Jul  3 04:16 /data/dbfile1/ib_logfile1
-rw-r-----. 1 mysql mysql 12M Aug  5 08:12 /data/dbfile1/ibtmp1
```
Σ( ° △ °|||)︴  改...改变了，唉，传言骗人啊~  什么放在内存中啊！ 全是骗子！

