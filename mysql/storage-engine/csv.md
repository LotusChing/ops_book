# CSV

`csv`几个特点

* csv是纯文本数据
* 不支持索引
* 每个列必须是`NOT NULL`



## 导入数据
建表
```
mysql> CREATE TABLE foo (i int not null, c char(10) not null) engine=csv;
```

拷贝已有`csv`替换默认`csv`
```
1,"FOOBAR"
2,"FOOBAZ"
3,"BARBAZ"
```

查看
```
mysql> select * from foo;
+---+--------+
| i | c      |
+---+--------+
| 1 | FOOBAR |
| 2 | FOOBAZ |
| 3 | BARBAZ |
+---+--------+
```
