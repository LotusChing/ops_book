# list


## 命令
* **RPUSH**: 创建列表键，从右端插入各元素(key不存在则创建并插入，存在则直接插入右端)
* **LPUSH**: 创建列表键，从右端插入各元素(key不存在则创建并插入，存在则直接插入左端)
* **LRANGE**: 获取列表键中指定范围的元素
* **LINDEX**: 获取列表键中指定下标的元素
* **LLEN**: 获取列表键长度(元素数量)
* **LPOP**: 删除列表键中最左端的元素
* **RPOP**: 删除列表键中最右端的元素
* **LREM**: 从左开始向右删除最多指定数量指定值的元素
* **LTRIM**: 裁剪列表键中元素
* **LSET**: 修改列表键中指定下标的元素
* ~~**BLPOP**: **没懂啥意思，有啥用**~~
* ~~**BRPOP**: **没懂啥意思，有啥用**~~


## 示例

**创建一个列表键，右端插入**
```
127.0.0.1:6379> RPUSH lst 1 2 3 4 5
(integer) 5
127.0.0.1:6379> LRANGE lst 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
```

**创建一个列表键，左端插入**
```
127.0.0.1:6379> LPUSH lst 1 2 3 4 5
(integer) 5
127.0.0.1:6379> LRANGE lst 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"
```


**获取列表键中指定下标的元素**
```
127.0.0.1:6379> LINDEX lst 0
"5"
```

**获取列表键的长度**
```
127.0.0.1:6379> LLEN lst
(integer) 5
```

**删除列表键中最左端的元素**
```
127.0.0.1:6379> LPOP lst
"5"
```

**删除列表键中最右端的元素**
```
127.0.0.1:6379> RPOP lst
"1"
```


**从左开始向右删除最多2个值为a的元素**
```
127.0.0.1:6379> RPUSH lst a a a b c d e f g
(integer) 9
127.0.0.1:6379> LRANGE lst 0 -1
1) "a"
2) "a"
3) "a"
4) "b"
5) "c"
6) "d"
7) "e"
8) "f"
9) "g"
127.0.0.1:6379> LREM lst 2 a
(integer) 2
127.0.0.1:6379> LRANGE lst 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
6) "f"
7) "g"
```

**裁剪列表键只保留指定范围的元素**
```
127.0.0.1:6379> RPUSH lst 1 2 3 4 5
(integer) 5
127.0.0.1:6379> LTRIM lst 0 2
OK
127.0.0.1:6379> LRANGE lst 0 -1
1) "1"
2) "2"
3) "3"
```

**修改列表中指定下标的元素**
```
127.0.0.1:6379> LSET lst 0 a
OK
127.0.0.1:6379> LRANGE lst 0 -1
1) "a"
2) "2"
3) "3"
```

## 时间复杂度
* **RPUSH**: O(n)，n为元素个数
* **LPUSH**: O(n)，n为元素个数
* **LRANGE**: O(s+n)，s是start_offset，n为元素个数
* **LINDEX**: O(n)，n为索引的偏移量
* **LLEN**: O(n)，n为元素个数
* **LPOP**: O(1)
* **RPOP**: O(1)
* **LREM**: O(n)，n为元素个数
* **LTRIM**: O(n)，n为元素个数
* **LSET**: O(n)，n为索引的偏移量
* ~~**BLPOP**: **没懂啥意思，有啥用**~~
* ~~**BRPOP**: **没懂啥意思，有啥用**~~

## 内部编码

* **ziplist**: 当列表键元素个数小于`list-max-ziplist-entries`(默认512)，并且各元素的值不超过`list-max-ziplist-value`(默认64)字节时，使用ziplist，ziplist会相对linkedlist更节省内存些
* **linkedlist**: 当列表键不满足ziplist条件时，redis将列表键转为linkedlist内部实现
* **quicklist**: 在redis 3.2中实现了一种结合ziplist和linkedlist各自优势的内部编码

    ```
    127.0.0.1:6379> OBJECT encoding lst
    "ziplist"
    127.0.0.1:6379> RPUSH lst "LotusChingLotusChingLotusChingLotusChingLotusChingLotusChingLotusChingLotusChing"
    (integer) 4
    127.0.0.1:6379> LRANGE lst 0 -1
    1) "a"
    2) "2"
    3) "3"
    4) "LotusChingLotusChingLotusChingLotusChingLotusChingLotusChingLotusChingLotusChing"
    127.0.0.1:6379> OBJECT encoding lst
    "linkedlist"
    ```

## 使用场景

### 灰度发布IP
```
127.0.0.1:6379> RPUSH gray_release_ip_list "192.168.1.150" "120.24.80.34"
(integer) 2
127.0.0.1:6379> LRANGE gray_release_ip_list 0 -1
1) "192.168.1.150"
2) "120.24.80.34"
```
 
## 栈/队列
lpush + lpop = Stack(栈)
lpush + rpop = Queue(队列)
