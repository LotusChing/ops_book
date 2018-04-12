# 日期

常见日期类型：


|类型|占用字节|表示范围|
|:--:|:-----:|:-----:|
|DATETIME|8|1000-01-01 00:00:00 ~ 9999:12:31 23:59:59|
|TIMESTAMP|4|1970-01-01 00:00:00 UTC ~ 2038-01-19 03:14:07 UTC|

```
mysql> create table test(
a bigint not null auto_increment primary key, 
b int not null, 
addTime timestamp not null default current_timestamp, 
updateTime datetime not null default current_timestamp on update current_timestamp);
```

