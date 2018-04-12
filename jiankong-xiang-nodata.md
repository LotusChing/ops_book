# 监控项nodata

## 从数据库入手
1. 从库中获取item信息
```sql
mysql> select itemid,name from items where name like '%queue size%';
```
2. 根据itemid查询库中有没有数据，clock字段是unix时间戳，格式化就可以确定时间了
```sql
mysql> select itemid,clock,value from history where itemid="28413" order by clock desc limit 3;
```

## 从日志入手
1. 检查zabbix-server日志
2. 检查zabbix-agent日志

