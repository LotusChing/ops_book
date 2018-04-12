# 字符串

|类型|说明|N的含义|是否有字符集|最大长度|
|:--:|:-:|:-----:|:--------:|:------:|
|CHAR(N)|定长字符|字符|是|255|
|VARCHAR(N)|变长字符|字符|是|16384|
|BINARY(N)|定长二进制字符|字节|否|255|
|VARBINARY(N)|变长二进制字符|字节|否|16384|
|TINYBLOB|二进制大对象|字节|否|256|
|BLOB|二进制大对象|字节|否|16k|
|MEDIUMBLOB|二进制大对象|字节|否|16M|
|LONGBLOB|二进制大对象|字节|否|4G|
|TINYTEXT|大对象|字节|是|256|
|TEXT|大对象|字节|是|16k|
|MEDIUMTEXT|大对象|字节|是|16M|
|LONGTEXT|大对象|字节|是|4G|

## 提醒
* BLOB、TEXT列创建索引时必须指定索引前缀长度
  ```
  mysql> create table  t4(a int primary key, b text,key(b));
 ERROR 1170 (42000): BLOB/TEXT column 'b' used in key specification without a key length
 
  mysql> create table  t4(a int primary key, b text,key(b(32)));
Query OK, 0 rows affected (0.09 sec)
  ```
* VARCHAR、VARBINARY前缀长度是可选的
* BLOB、TEXT不能有默认值
* BLOB、TEXT列排序时只能使用该列的前`max_sort_length`字节


## 字符集
* character set: a set of symbols and encodings
* 常见：utf8, utf8mb4, gbk, gb18030
* ci: 忽略大小写 
 * 使用`utf8_bin`、`utf8mb4_bin`来区分大小写

查看支持的字符集
```
mysql> show charset;
```




## 诡问
### N代表什么？

###char(10)插入中文"刘聪"会填充什么？






