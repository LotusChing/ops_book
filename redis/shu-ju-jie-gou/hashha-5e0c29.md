# hash(哈希|字典)

* **HSET**: 创建哈希键并设置一个field
* **HGET**: 获取哈希键中的field值
* **HDEL**: 删除哈希键中的field
* **HKEYS**: 获取哈希键中所有的field
* **HVALS**: 获取哈希键中的所有value
* **HLEN**: 获取哈希键中field数量
* **HMSET**: 批量设置哈希键的field
* **HMGET**: 批量获取哈希键中的filed值
* **HGETALL**: 获取所有哈希键的field和value
* **HEXISTS**: 判断哈希键中是否有指定field
* **HSTRLEN**: 获取哈希键中某field的长度(string)，**3.2版本以后支持**


## 常用示例

**创建哈希类型的键值**

```
127.0.0.1:6379> HSET user name LotusChing 
(integer) 1
127.0.0.1:6379> HSET user age 21
(integer) 1
127.0.0.1:6379> HSET user gender "Male"
(integer) 1
```

HSET 不支持创建一次性创建多field
```
127.0.0.1:6379> HSET user name "LotusChing" age 21
(error) ERR wrong number of arguments for 'hset' command
```

**获取哈希键中的field值**
```
127.0.0.1:6379> HGET user name
"LotusChing"
127.0.0.1:6379> hget user age
"21"
127.0.0.1:6379> hget user gender
"Male"
```

HGET 不支持一次获取多个field

**获取哈希键中的fields**
```
127.0.0.1:6379> HKEYS user
1) "name"
2) "age"
```

**获取哈希键中的所有field的value**
```
127.0.0.1:6379> HVALS user
1) "LotusChing"
2) "21"
```

**删除哈希键中某个field**
```
127.0.0.1:6379> HDEL user age
(integer) 1
127.0.0.1:6379> HKEYS user
1) "name"
```

**统计哈希中field的个数**
```
127.0.0.1:6379> HKEYS user
1) "name"
2) "age"
3) "gender"
127.0.0.1:6379> HLEN user
(integer) 3
```

**批量设置哈希键的field**
```
127.0.0.1:6379> HMSET user name "LotusChing" age 21 gender "Male"
OK
127.0.0.1:6379> HKEYS user
1) "name"
2) "age"
3) "gender"
127.0.0.1:6379> HVALS user
1) "LotusChing"
2) "21"
3) "Male"
```

**批量获取哈希键中field的value**
```
127.0.0.1:6379> HMGET user name age gender
1) "LotusChing"
2) "21"
3) "Male"
127.0.0.1:6379> HMGET user name age
1) "LotusChing"
2) "21"
```

**判断哈希键中field是否存在**
```
127.0.0.1:6379> HEXISTS user name
(integer) 1
127.0.0.1:6379> HEXISTS user hobbies
(integer) 0
```

**一次性获取哈希键中所有的fields和values**

PS: 尽量避免使用`HGETALL`，因为如果哈希键field过多的话，可能会导致Redis阻塞，建议使用`HMGET`获取所需哈希键中的field值，或者采用`HSCAN`
```
127.0.0.1:6379> HGETALL user
1) "name"
2) "LotusChing"
3) "age"
4) "21"
5) "gender"
6) "Male"
```

## 时间复杂度
* **HSET**: O(1)
* **HGET**: O(1)
* **HDEL**: O(n)，n为field的数量
* **HKEYS**: O(n)，n为field的数量
* **HVALS**: O(n)，n为field的数量
* **HLEN**: O(1)
* **HMSET**: O(n)，n为field的数量
* **HMGET**: O(n)，n为field的数量
* **HGETALL**: O(n)，n为field的数量
* **HEXISTS**:  O(1)
* **HSTRLEN**: O(1)，**3.2版本以后支持**


## 内部编码
* **ziplist**: 当哈希键中的field小于`hash-max-ziplist-entries`(默认512)时，并且哈希键中的所有value小于`hash-max-ziplist-value`(64)时，使用ziplist，这种类型使用更紧凑的结构实现多个元素的存储，相对hashtable来说更节省内存开销
* **hashtable**: 当哈希键无法满足ziplist的条件时，会将哈希键内部编码转为hastable，因为此时使用ziplist读写效率会下降，而hashtable更为适合，hashtable的时间复杂度O(1)

    
    ```
    127.0.0.1:6379> HMSET user name "LotusChing" age 21 gender "Male"
    OK
    127.0.0.1:6379> OBJECT encoding user
    "ziplist"
    
    
    127.0.0.1:6379> HMSET user name "LotusChingLotusChingLotusChingLotusChingLotusChingLotusChingLotusChingLotusChing" age 21 gender "Male"
    OK
    127.0.0.1:6379> OBJECT encoding user
    "hashtable"
    ```
    
# 使用场景

## 缓存
利用一个哈希键存储多条缓存信息，例如存储json序列化后的用户信息
```
HSET user_info_cache user_id_info "{'id': 1, 'name': 'LotusChing', 'age': 21, 'gender': 'Male'}"
```
