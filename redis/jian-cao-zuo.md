# 键操作

## 命令
* KEYS
* DBSIZE
* EXISTS
* DEL
* SET
* TYPE
* RENAME
* RANDOMKEY
* EXPIRE
* TTL
* PTTL
* EXPIREAT
* MOVE
* DUMP
* RESTORE
* MIGRATE
* SCAN
* HSCAN
* SSCAN
* ZSCAN
* SELECT
* FLASHDB
* FLASHALL

## 常用示例


**列出所有的键**
```
127.0.0.1:6379> keys *
1) "name"
```

**统计键的数量**
```
127.0.0.1:6379> DBSIZE
(integer) 1
```

**键是否存在**
1为True，0为False
```
127.0.0.1:6379> EXISTS name
(integer) 1
127.0.0.1:6379> EXISTS gender
(integer) 0
```

**删除键**
删除多个键以空格
```
127.0.0.1:6379> del name
(integer) 1
```

删除多个key，PS: 先创建三个key
```
127.0.0.1:6379> set name "LotusChing"
OK
127.0.0.1:6379> set age 21
OK
127.0.0.1:6379> set gender "male"
OK

127.0.0.1:6379> keys *
1) "age"
2) "gender"
3) "name"

127.0.0.1:6379> del name age gender
(integer) 3
```

**获取键的类型**
通过`type`可以获取键的数据类型，键不存在的返回None
```
127.0.0.1:6379> set name "LotusChing"
OK
127.0.0.1:6379> type name
string
127.0.0.1:6379> 
127.0.0.1:6379> set age 21
OK
127.0.0.1:6379> type age
string
127.0.0.1:6379> RPUSH mylist a b c d e
(integer) 5
127.0.0.1:6379> type mylist
list
127.0.0.1:6379> type gender
none
```

**重命名键名，如果重名则替换**

在执行重命名时，如果KEY对应的值过大，有可能导致Redis阻塞

```
127.0.0.1:6379> set username "LotusChing"
OK
127.0.0.1:6379> get username
"LotusChing"
127.0.0.1:6379> RENAME username user_name
OK
127.0.0.1:6379> keys *
1) "user_name"
127.0.0.1:6379> get user_name
"LotusChing"
```

**重命名键名，如果重名则取消**

在执行重命名时，如果KEY对应的值过大，有可能导致Redis阻塞

```
127.0.0.1:6379> set username "LotusChing"
OK
127.0.0.1:6379> set user_name "Da"
OK
127.0.0.1:6379> RENAMENX username user_name
(integer) 0
127.0.0.1:6379> RENAMENX username user_name1
(integer) 1
127.0.0.1:6379> keys *
1) "user_name1"
2) "user_name"
```


**随即返回一个KEY**
```
127.0.0.1:6379> RANDOMKEY 
"city"
127.0.0.1:6379> RANDOMKEY 
"user_name"
127.0.0.1:6379> RANDOMKEY 
"city"
127.0.0.1:6379> RANDOMKEY 
"user_name"
```

### 键过期

Redis不支持对二级数据结构设置过期时间，例如列表中的某个元素，或者哈希中的某个field

**设置键过期秒级时间**
设置name这个key的过期时间是10秒钟，超过十秒钟redis会自动删除这个过期的key
```
127.0.0.1:6379> set name "LotusChing"
OK
127.0.0.1:6379> EXPIRE name 10
(integer) 1
```

**查看键的剩余过期时间**

如果返回值大于0则为键的剩余时间，如果返回值为`-1`为键为设置过期时间，如果为`-2`为键不存在

```
127.0.0.1:6379> TTL name
(integer) 8
127.0.0.1:6379> TTL name
(integer) 7
127.0.0.1:6379> TTL name
(integer) 6
127.0.0.1:6379> TTL name
(integer) 1
127.0.0.1:6379> TTL name
(integer) 0
127.0.0.1:6379> TTL name
(integer) -2
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> keys *
(empty list or set)
```


**根据时间戳设置键过期时间，查看剩余过期时间(毫秒)**
其实设置过期时间时无论是秒级还是毫秒级，最终Redis内部都会使用`EXPIREAT`时间戳的形式设置
```
127.0.0.1:6379> set name "Da"
OK
127.0.0.1:6379> EXPIREAT name 1514353185
(integer) 1
127.0.0.1:6379> ttl name
(integer) 100
127.0.0.1:6379> ttl name
(integer) 99
127.0.0.1:6379> pttl name
(integer) 96256
127.0.0.1:6379> pttl name
(integer) 95518
127.0.0.1:6379> pttl name
(integer) 94892
127.0.0.1:6379> pttl name
(integer) 94065
```

**清除键的过期时间设置**
```
127.0.0.1:6379> set name "Da"
OK
127.0.0.1:6379> expire name 60
(integer) 1
127.0.0.1:6379> ttl name
(integer) 57
127.0.0.1:6379> PERSIST name
(integer) 1
127.0.0.1:6379> ttl name
(integer) -1
```

除了`PERSIST`外，set时也会清除掉过期时间的设置
```
127.0.0.1:6379> set name "Da"
OK
127.0.0.1:6379> ttl name
(integer) -1
127.0.0.1:6379> EXPIRE name 60
(integer) 1
127.0.0.1:6379> ttl name
(integer) 57
127.0.0.1:6379> ttl name
(integer) 56
127.0.0.1:6379> set name "Da"
OK
127.0.0.1:6379> ttl name
(integer) -1
```

### 迁移键
Redis迁移键主要有三种方式，`MOVE`、`DUMP + RESTORE`、`MIGRATE`

MOVE是迁移key到另一个库
DUMP是序列化键后，在目标通过RESTORE反序列化并导入
MIGRATE是常规意义上的迁移

**通过DUMP序列化键**
```
127.0.0.1:6379> set user "LotusChing"
OK
127.0.0.1:6379> DUMP user
"\x00\nLotusChing\x06\x00\xcfj\xc3e<\xa8\xd4?"
```

**通过RESTORE反序列化创建一个键**
```
127.0.0.1:6379> RESTORE user 0 "\x00\nLotusChing\x06\x00\xcfj\xc3e<\xa8\xd4?"
OK
127.0.0.1:6379> get user
"LotusChing"
```



**通过MIGRATE迁移一个键**

6379实例

```
127.0.0.1:6379> keys *
1) "user_name1"
2) "user"
3) "name"
4) "user_name"
127.0.0.1:6379> MIGRATE 127.0.0.1 6380 user_name1 0 1000
OK
127.0.0.1:6379> keys *
1) "user"
2) "name"
3) "user_name"
```

6380实例
```
127.0.0.1:6380> keys *
1) "user_name1"
127.0.0.1:6380> get user_name1
"LotusChing"
```

**通过MIGRATE迁移多个键，并且不删除源实例中的键**
6379实例
```
127.0.0.1:6379> MIGRATE 127.0.0.1 6380 "" 0 1000 COPY keys user name
OK
127.0.0.1:6379> keys *
1) "user"
2) "name"
3) "user_name"
```

6380实例
```
127.0.0.1:6380> keys *
1) "user"
2) "user_name1"
3) "name"
```

## 键遍历

**通过KEYS遍历键**
```
127.0.0.1:6380> keys *
1) "user"
2) "user_name1"
3) "name"
```

**通过SCAN进行遍历键**

为什么使用SCAN遍历键，因为KEYS过多时，执行KEYS * 会产生阻塞的问题，而SCAN是渐进式遍历，通过多次遍历完成遍历一定量的键，最终实现遍历全部键，有点像是Python中的Generator

SCAN从0开始，默认返回下一次的游标和10个KEY，如果下一次游标为0，则表示遍历完所有的键，
```
127.0.0.1:6380> mset a 1 b 1 c 1 d 1 e 1 f 1 g 1 h 1 i 1 j 1 k 1 l 1 m 1 n 1 o 1 p 1 q 1 r 1 s 1 t 1 u 1 v 1 w 1 x 1 y 1 z 1
OK
127.0.0.1:6380> keys *
 1) "a"
 2) "i"
 3) "user_name1"
 4) "y"
 5) "g"
 6) "p"
 7) "f"
 8) "user"
 9) "k"
10) "u"
11) "m"
12) "x"
13) "w"
14) "d"
15) "c"
16) "v"
17) "t"
18) "z"
19) "o"
20) "b"
21) "r"
22) "q"
23) "name"
24) "n"
25) "s"
26) "j"
27) "e"
28) "h"
29) "l"
127.0.0.1:6380> SCAN 0
1) "6"
2)  1) "d"
    2) "c"
    3) "o"
    4) "u"
    5) "m"
    6) "e"
    7) "user_name1"
    8) "z"
    9) "s"
   10) "j"
127.0.0.1:6380> SCAN 6
1) "13"
2)  1) "p"
    2) "f"
    3) "user"
    4) "a"
    5) "i"
    6) "v"
    7) "t"
    8) "n"
    9) "g"
   10) "b"
127.0.0.1:6380> SCAN 13
1) "0"
2) 1) "x"
   2) "h"
   3) "y"
   4) "k"
   5) "r"
   6) "q"
   7) "name"
   8) "w"
   9) "l"
127.0.0.1:6380> 
```
除了KEYS会带来Redis阻塞的问题，HGETALL、SMEMBERS、ZRANGE在数据量大时也都有带来Redis阻塞的可能，对应的解决思路是HSACN、SSCAN、ZSCAN

## 数据库管理

Redis默认16个DB，0~16，默认是0库

**切换数据库**
```
127.0.0.1:6380> select 1
OK
127.0.0.1:6380[1]> set a 1
OK
127.0.0.1:6380[1]> keys *
1) "a"
127.0.0.1:6380[1]> select 2
OK
127.0.0.1:6380[2]> keys *
(empty list or set)
```

**清空当前库中所有键**
```
127.0.0.1:6380[2]> select 1
OK
127.0.0.1:6380[1]> keys *
1) "a"
127.0.0.1:6380[1]> set b 2
OK
127.0.0.1:6380[1]> keys *
1) "b"
2) "a"
127.0.0.1:6380[1]> FLUSHDB
OK
127.0.0.1:6380[1]> keys *
(empty list or set)
```

**清空所有库所有键
```
127.0.0.1:6380[1]> FLUSHALL
OK
127.0.0.1:6380> keys *
(empty list or set)
```

**尽量不要将数据分开存放在多个库**
1. Redis是单线程，即便使用多个库，但都是在一个CPU上，彼此间还是会受到影响
2. 多数据存放导致在多业务环境下，一个业务的慢查询，将会影响其他业务方运维调试和问题定位很麻烦
3. 部分客户端不支持这种方式，即便支持来回切数字很容易弄乱


**建议配置多Redis实例**
1. 解决了业务数据分离的问题
2. 充分利用到了其他CPU核心，避免资源浪费