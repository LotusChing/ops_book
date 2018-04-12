# 字符串

## 设置语法
```
set key value [EX seconds] [PX ms] [nx|xx]
```

* key: 键名
* value: 键值
* ex seconds: 键秒级过期时间
* ex ms: 键毫秒及过期时间
* nx: 键不存在才能设置，setnx和nx选项作用一样，用于添加，分布式锁的实现
* xx: 键存在才能设置，setxx和xx选项作用一样，用于更新

## 示例

**创建简单的键值**
```
127.0.0.1:6379> set gender "male" 
OK
127.0.0.1:6379> get gender
"male"
```

**创建键值并设置10秒过期时间**

通过EX参数
```
127.0.0.1:6379> set gender "male" EX 100
OK
```

通过PX参数
```
127.0.0.1:6379> set gender "male" PX 10000
OK
127.0.0.1:6379> ttl gender
(integer) 9
```

**指定键不存在时创建键值**
```
127.0.0.1:6379> set gender "male" NX
OK
127.0.0.1:6379> set gender "male1" NX
(nil)
```

**指定键存在时创建键值**
```
127.0.0.1:6379> set gender "male1" XX
OK
127.0.0.1:6379> get gender
"male1"
```

**获取值的一部分(字符串切片)**
```
127.0.0.1:6379> GET name 
"LotusChing"
127.0.0.1:6379> GETRANGE name 0 4
"Lotus"
127.0.0.1:6379> GETRANGE name 0 -1
"LotusChing"
```

**批量获取多个键的值**
```
127.0.0.1:6379> mget name age gender
1) "LotusChing"
2) "21"
3) "male1"
```

**批量设置多个键值**
批量设置值成功了，但是参数好像没有生效
```
127.0.0.1:6379> mset name "Da" EX 100 age 21 gender "male" EX 200
OK
127.0.0.1:6379> ttl name
(integer) -1
127.0.0.1:6379> get name
"Da"
127.0.0.1:6379> get age
"21"
127.0.0.1:6379> get gender
"male"
```

书中提到一个有趣的概念，批量操作可以提供效率节省时间

逐条get/set的时间消耗公式：

`n次get/set时间 = n次网络时间 + n次命令时间`

批量get/set的时间消耗公式：
`n次get/set时间 = 1次网络时间 + n次命令时间`

合理的使用批量操作可以提高Redis性能，但是注意不要量太大，**如果过量的话可能会导致Redis阻塞**


**对键追加值**
```
127.0.0.1:6379> get name
"Da"
127.0.0.1:6379> APPEND name "Yo"
(integer) 4
127.0.0.1:6379> get name
"DaYo"
```

**获取键值长度**
```
127.0.0.1:6379> strlen name
(integer) 4
```

## 时间复杂度
* set: O(1)
* get: O(1)
* del: O(k)，k为键的个数
* mget: O(k)，k为键的个数
* mset: O(k)，k为键的个数
* append: O(1)
* str: O(1)
* getrange: O(n), n为字符串的长度

## 内部编码
* int: 8字节长整型
* embstr: 小于39字节值
* raw: 大于39字节的值

## 典型场景
**1. 缓存**

**2. 共享Session**

**3. 限速(计数+超时)**
