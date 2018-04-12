# GRANT
## 给权限
之前一直是图省事，直接 `all on db.*`，后来仔细看了才知道`MySQL`授权还是挺厉害的，粒度还是比较细的，可以做到针对每个column权限控制
### 内容限制
#### 只能读某些列
这里限制了`u2`用户只能读取`name, age`列
```
mysql> grant SELECT (name,age) on da.t2 to u2@'127.0.0.1' identified by 'u2';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

**测试结果：**
```
mysql> select * from t2;
ERROR 1143 (42000): SELECT command denied to user 'u2'@'localhost' for column 'id' in table 't2'
mysql> select name, age from t2;
+------+------+
| name | age  |
+------+------+
| Yo   |   21 |
+------+------+
1 row in set (0.00 sec)
```


#### 只读/只改某些列
在实验完只读指定列时，脑子里就闪过，能否支持除了限制读取指定列，在限制下update的列，试了下，果然可以，`MySQL`棒棒哒

```
mysql> grant SELECT (name,age), UPDATE(name) on da.t2 to u3@'127.0.0.1' identified by 'u3';
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

**测试结果：**

读取
```
mysql> select * from t2;
ERROR 1143 (42000): SELECT command denied to user 'u3'@'localhost' for column 'id' in table 't2'
mysql> select name,age from t2;
+------+------+
| name | age  |
+------+------+
| Yo   |   21 |
+------+------+
1 row in set (0.00 sec)
```

修改
```
mysql> update t2 set age=18;
ERROR 1143 (42000): UPDATE command denied to user 'u3'@'localhost' for column 'age' in table 't2'
mysql> update t2 set name='Yo';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

### 访问限制
`MySQL`支持限制用户源`ip`或者网段，或者域名(不常用，而且我也暂时没想明白意义)，上面几个实验都是授权本机`127.0.0.1`，也可以支持授权网段

```
# mysql -uu4 -pu4 -h127.0.0.1 -sNe "show databases;"
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1045 (28000): Access denied for user 'u4'@'127.0.0.1' (using password: YES)
[root@Da ~]# mysql -uu4 -pu4 -h192.168.2.30 -sNe "show databases;"
mysql: [Warning] Using a password on the command line interface can be insecure.
information_schema
da
test
```


### 频率限制
MySQL支持访问频率限制，但是粒度比较粗，如果将来能够支持到秒级别就很棒了，`MySQL`现在支持每小时select、每小时update，最大连接数，用户最大连接数
```
mysql> grant SELECT on da.t2 to u7@'127.0.0.1' identified by 'u7' with MAX_QUERIES_PER_HOUR 6;
```


测试结果：
```
mysql> select * from t2;
+------+------+------+--------+
| id   | name | age  | gender |
+------+------+------+--------+
|    1 | Yo   |   21 | Man    |
+------+------+------+--------+
1 row in set (0.00 sec)

mysql> select * from t2;
+------+------+------+--------+
| id   | name | age  | gender |
+------+------+------+--------+
|    1 | Yo   |   21 | Man    |
+------+------+------+--------+
1 row in set (0.00 sec)

mysql> select * from t2;
+------+------+------+--------+
| id   | name | age  | gender |
+------+------+------+--------+
|    1 | Yo   |   21 | Man    |
+------+------+------+--------+
1 row in set (0.00 sec)

mysql> select * from t2;
ERROR 1226 (42000): User 'u7' has exceeded the 'max_questions' resource (current value: 6)
mysql> select * from t2;
ERROR 1226 (42000): User 'u7' has exceeded the 'max_questions' resource (current value: 6)
```

看样子默认mysql client在连接上mysql服务器后会执行一些初始化的sql~   

![](/assets/(}I{3O7VU0LXCC$EI5B1R4S.png)


## 查看用户权限
```
mysql> show grants for u8@'127.0.0.1';
+-----------------------------------------------+
| Grants for u8@127.0.0.1                       |
+-----------------------------------------------+
| GRANT USAGE ON *.* TO 'u8'@'127.0.0.1'        |
| GRANT SELECT ON `da`.`t2` TO 'u8'@'127.0.0.1' |
+-----------------------------------------------+
```
`USAGE`表示能够能够登录，下面的那个条才是`u8`它的权限

