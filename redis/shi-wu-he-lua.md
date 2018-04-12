# 事务和Lua

# 事务

要么全执行要么全不执行

**Redis简单事务示例**

MULTI: 代表开启一个事务，类似MySQL中的start
QUEUED: 代表命令被加入到队列中等待执行
EXEC: 代表开始执行事务中的命令

```
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SADD user_yo_follows da
QUEUED
127.0.0.1:6379> SADD user_da_fans yo
QUEUED
127.0.0.1:6379> EXEC
```

在事务提交前，是查看不到事务队列中的数据的


**如果事务中某个命令语法错误，在EXEC时会提示Redis已经因为错误而放弃掉该事务**
```
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set name "Da"
QUEUED
127.0.0.1:6379> set name"Yo"
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379> EXEC
(error) EXECABORT Transaction discarded because of previous errors.
```

**如何确保事务提交前操作的键被另外会话修改？**

通过watch实现，如果事务未提交前，事务内的有一个键被修改了，事务提交后所有都操作都不执行，

终端一：
```
127.0.0.1:6379:T1> set s1 "i love "
OK
127.0.0.1:6379:T2> watch s1
OK
127.0.0.1:6379:T3> multi 
OK
```


终端二：
```
127.0.0.1:6379:T4> APPEND s1 "Python"
(integer) 13
```

终端一：
```
127.0.0.1:6379:T5> append s1 " Shell"
QUEUED
127.0.0.1:6379:T6> set s2 "i hate math"
QUEUED
127.0.0.1:6379:T7> EXEC
(nil)
127.0.0.1:6379:T8> keys *
1) "s1"
127.0.0.1:6379:T9> get s1
"i love Python"
```

## Lua

Lua语法不再记录，这里只记录下几个小例子

### eval
**1. 给列表中每隔字符串键自增**
```
root@iZ947mgy3c5Z:~# cat user_ratio_incr.lua
local user_ratio_list = redis.call("lrange", KEYS[1], 0, -1)
local count = 0
for index, key in ipairs(user_ratio_list)
do
     redis.call("incr", key)
     count = count + 1
end
return count
```

使用方法
```
redis-cli --eval user_ratio_incr.lua 列表键
```


**2. 批量创建键**
```
root@iZ947mgy3c5Z:~# cat create_100_key.lua
for i=1, 20
do
    local key="test_" .. i
    redis.call("set", key, i)
end
```

使用方法
```
redis-cli --eval create_100_key.lua 
```

**PS: print()输出永远都是nil，建议使用lua+redis模块的方式操作**

### lua client + redis

**安装luarocks**
```
apt-get install luarocks -y
```

**通过luarocks安装redis-lua**
```
luarocks install redis-lua
```

**测试**
```
root@iZ947mgy3c5Z:/tmp# cat test.lua 
pcall(require, "luarocks.require")

local redis = require 'redis'

local params = {
    host = '127.0.0.1',
    port = 6379,
}

local client = redis.connect(params)
local value = client:get('user')

print(value)
root@iZ947mgy3c5Z:/tmp# lua test.lua 
LotusChing
```

## redis管理脚本

* script load
* script exist
* script flush
* script kill

以这个批量创建key的lua脚本为例
```
root@iZ947mgy3c5Z:~# cat create_5_key.lua 
for i=1, 5
do
    local key = "test_" .. i
    redis.call("set", key, i)
end
```

这个load是把脚本内容载入到Redis内存中，以避免多次传输脚本消耗带宽，load会返回一个SHA1值
```
root@iZ947mgy3c5Z:~# redis-cli 
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> 
root@iZ947mgy3c5Z:~# redis-cli script load "$(cat create_5_key.lua)"
"8694ef1bcb6d637bc7a23dbd991d97b23b82b1c7"
root@iZ947mgy3c5Z:~# redis-cli 
127.0.0.1:6379> SCRIPT EXISTS "8694ef1bcb6d637bc7a23dbd991d97b23b82b1c7"
1) (integer) 1
127.0.0.1:6379> EVALSHA "8694ef1bcb6d637bc7a23dbd991d97b23b82b1c7" 0
(nil)
127.0.0.1:6379> keys *
1) "test_4"
2) "test_5"
3) "test_1"
4) "test_2"
5) "test_3"
127.0.0.1:6379> SCRIPT EXISTS "8694ef1bcb6d637bc7a23dbd991d97b23b82b1c7"
1) (integer) 1
127.0.0.1:6379> SCRIPT FLUSH 
OK
127.0.0.1:6379> SCRIPT EXISTS "8694ef1bcb6d637bc7a23dbd991d97b23b82b1c7"
1) (integer) 0
```


