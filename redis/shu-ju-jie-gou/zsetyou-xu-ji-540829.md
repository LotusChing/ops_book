# zset


## 命令
* ZADD:
* ZRANGE:
* ZREVRANGE:
* ZCARD:
* ZSCORE:
* ZRANK:
* ZREVRANK:
* ZINCRBY:
* ZREVRANGE:
* ZRANGEBYSCORE:
* ZCOUNT:
* ZREM:
* ZREMBYSCORE:
* ZREMBYRANK:
* ZINTERSTORE:
* ZUNIONSTORE:
* ~~ZDIFFSTORE: 不支持计算差集~~


## 示例

**创建一个有序集合键**
```
127.0.0.1:6379> ZADD zset1 60 "Da" 80 "LotusChing" 90 "Yo"
(integer) 1
```


**获取有序集合键中的顺序排名**
    ```
    127.0.0.1:6379> ZRANGE zset1 0 -1
    1) "Da"
    2) "LotusChing"
    3) "Yo"
    
    127.0.0.1:6379> ZRANGE zset1 0 -1 withscores
    1) "Da"
    2) "60"
    3) "LotusChing"
    4) "80"
    5) "Yo"
    6) "90"
    ```

**获取有序集合键中的倒叙排名情况**
    ```
    127.0.0.1:6379> ZREVRANGE zset1 0 -1
    1) "Yo"
    2) "LotusChing"
    3) "Da"
    
    127.0.0.1:6379> ZREVRANGE zset1 0 -1 withscores
    1) "Yo"
    2) "90"
    3) "LotusChing"
    4) "80"
    5) "Da"
    6) "60"
    ```

**获取有序集合键的元素数量**
```
127.0.0.1:6379> ZCARD zset1
(integer) 3
```

**获取元素成员在有序集合键中的顺序排名**
```
127.0.0.1:6379> ZRANK zset1 Yo
(integer) 2
```

**获取元素成员在有序集合键中的倒序排名**
```
127.0.0.1:6379> ZREVRANK zset1 Yo
(integer) 0
```

**获取有序集合键中指定元素的分数**
```
127.0.0.1:6379> ZSCORE zset1 Yo
"90"
```

**增加有序集合键中元素的分数**
```
127.0.0.1:6379> ZINCRBY zset1 10 Da
"70"
127.0.0.1:6379> ZSCORE zset1 Da
"70"
```

**获取分数在指定范围内的元素**
```
127.0.0.1:6379> ZRANGEBYSCORE zset1 0 80  withscores
1) "Da"
2) "70"
3) "LotusChing"
4) "80"
```

`-inf`是无限小，`+inf`是无限大
```
127.0.0.1:6379> ZRANGEBYSCORE zset1 -inf +inf  withscores
1) "Da"
2) "70"
3) "LotusChing"
4) "80"
5) "Yo"
6) "90"
```




## 内部编码
* ziplist: 当有序集合键中的元素数量小于`zset-max-ziplist-entries`(默认128)时，并且所有元素值不超过`zset-max-ziplist-value`(默认64)字节时，使用ziplist作为内部编码
* skiplist: 当有序集合键不满足ziplist的条件时，使用skiplist作为内部编码以减少此时ziplist读写效率下降的问题


## 应用场景
### 排行榜系统
### 点赞