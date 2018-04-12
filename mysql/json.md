# Json

5.7之前MySQL并不支持Json，大多数都是应用端将数据库转为Json，然后存进BLOB类型的字段里

5.7出来后，MySQL支持了Json，同时带来了一下好处

* Json有效性检查
* 查询性能提升，因为我们可以或许Json里指定数据，而不必遍历全部
* 通过虚拟列实现对Json部分数据进行索引


## 建表
```
mysql> create table t7 (id int auto_increment primary key, data json );
Query OK, 0 rows affected (0.13 sec)

mysql> desc t7;
+-------+---------+------+-----+---------+----------------+
| Field | Type    | Null | Key | Default | Extra          |
+-------+---------+------+-----+---------+----------------+
| id    | int(11) | NO   | PRI | NULL    | auto_increment |
| data  | json    | YES  |     | NULL    |                |
+-------+---------+------+-----+---------+----------------+
2 rows in set (0.02 sec)
```

随意插入一些Json数据
```
mysql> insert into t7(data) values('{"name":"Da","age":21}');
mysql> insert into t7(data) values('{"name": "Yo", "age": 20}');
```


## Json函数
### JSON_EXTRACT(GET)
获取Json获取指定字段的数据
```
mysql> select JSON_EXTRACT(data, '$.name') from t7;
+------------------------------+
| JSON_EXTRACT(data, '$.name') |
+------------------------------+
| "Da"                         |
+------------------------------+
```

### JSON_INSERT(SET)
插入一个字段与数据到Json中

所有记录
```
mysql> update t7 set data = json_insert(data, '$.gender', 'Male');
```

指定记录
```
mysql> update t7 set data = json_insert(data, '$.address2', 'Shanghai') where json_extract(data, '$.name')='Da';

mysql> select * from t7;
+----+-------------------------------------------------------------------------------------------+
| id | data                                                                                      |
+----+-------------------------------------------------------------------------------------------+
|  2 | {"age": 21, "name": "Da", "gender": "Male", "address": "BeiJing", "address2": "Shanghai"} |
|  3 | {"age": 20, "name": "Yo", "gender": "Male", "address": "BeiJing"}                         |
+----+-------------------------------------------------------------------------------------------+
```

### JSON_MERGE(GET)
SELECT查询时，将多个字段的值合并为列表，
```
mysql> select id,JSON_EXTRACT(data, '$.name'), JSON_MERGE(JSON_EXTRACT(data, '$.address'), JSON_EXTRACT(data, '$.address2')) as address_info from t7;;
+----+------------------------------+-------------------------+
| id | JSON_EXTRACT(data, '$.name') | address_info            |
+----+------------------------------+-------------------------+
|  2 | "Da"                         | ["BeiJing", "Shanghai"] |
|  3 | "Yo"                         | NULL                    |
+----+------------------------------+-------------------------+
```
由于`Yo`的`Json`数据中只有`address`，没有`address2`所以返回了`NULL`

解决这个问题很简答，加个判断就行了
```
mysql> select id,JSON_EXTRACT(data, '$.name'), JSON_MERGE(JSON_EXTRACT(data, '$.address'), JSON_EXTRACT(data, '$.address2')) as address_info from t7 where JSON_EXTRACT(data, '$.address2') IS NOT NULL;
```


### JSON_ARRAY_APPEND(SET)
修改合并字段值为列表
```
mysql> update t7 set data=JSON_ARRAY_APPEND(data, '$.address', JSON_EXTRACT(data, '$.address2')) where JSON_EXTRACT(data, '$.address2') IS NOT NULL; 
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from t7;
+----+---------------------------------------------------------------------------------------------------------+
| id | data                                                                                                    |
+----+---------------------------------------------------------------------------------------------------------+
|  2 | {"age": 21, "name": "Da", "gender": "Male", "address": ["BeiJing", "Shanghai"], "address2": "Shanghai"} |
|  3 | {"age": 20, "name": "Yo", "gender": "Male", "address": "BeiJing"}                                       |
+----+---------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

也可以直接插入一个值
```
mysql> update t7 set data=JSON_ARRAY_APPEND(data, '$.address', 'ChengDu') where JSON_EXTRACT(data, '$.address2') IS NULL;
```


### JSON_REMOVE(SET)
删除Json指定字段
```
mysql> update t7 set data=JSON_REMOVE(data, '$.address2')
```

