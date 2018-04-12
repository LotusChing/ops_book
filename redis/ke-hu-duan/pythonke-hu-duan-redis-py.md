# redis-py

## 安装
```
pip install redis
```

## 简单使用
基本的SET、GET
```python
root@iZ947mgy3c5Z:/tmp# cat rds_1.py 
import redis

client = redis.StrictRedis(host="127.0.0.1", port=6379)
key="name"
value="LotusChing"
set_res = client.set(key, value)
if set_res:
    get_res = client.get(key)
    print('Key: {}, Value: {}'.format(key, get_res.decode()))
```

执行结果
```
root@iZ947mgy3c5Z:/tmp# python3.5 rds_1.py 
Key: name, Value: LotusChing
```

## 五种数据结构
初始化client对象
```
import redis
client = redis.StrictRedis(host="127.0.0.1", port=6379)
```

**字符串(string)**
```python
>>> client.set("tag", "XX")
True
>>> client.get("tag")
b'XX'
>>> client.incr("count")
1
```

**哈希(hash|dict)**
```
>>> client.hset("user_info", "name", "Da")
1
>>> client.hset("user_info", "age", 21)
1
>>> client.hgetall("user_info")
{b'name': b'Da', b'age': b'21'}
```


**列表(list)**
```
>>> client.rpush("lst", "1", "2")
2
>>> client.rpush("lst", "3", "4", "5")
5
>>> client.lrange("lst", 0, -1)
[b'1', b'2', b'3', b'4', b'5']
```

**集合(set)**
```
>>> client.sadd("myset1", "1", "2", "3", "Da")
4
>>> client.sadd("myset1", "Yo")
1
>>> client.smembers("myset1")
{b'Yo', b'1', b'Da', b'3', b'2'}
```

**有序集合(zset)**
```
>>> client.zadd("myset2", "80", "Da", "95", "Yo", "100", "LotusChing")
3
>>> client.zrange("myset2", 0, -1)
[b'Da', b'Yo', b'LotusChing']
>>> client.zrange("myset2", 0, -1, withscores=True)
[(b'Da', 80.0), (b'Yo', 95.0), (b'LotusChing', 100.0)]
```

## Pipeline
测试py脚本
```python
root@iZ947mgy3c5Z:/tmp# cat rds_2.py 
import redis
client = redis.StrictRedis(host="127.0.0.1", port=6379)
pipeline = client.pipeline(transaction=False)

pipeline.set("hello", "world")
pipeline.incr("counter")
result = pipeline.execute()
print("Pipeline Result: {}".format(result))
```

执行结果
```
root@iZ947mgy3c5Z:/tmp# python3.5 rds_2.py 
Pipeline Result: [True, 1]
```

查看Redis
```
127.0.0.1:6379> keys *
1) "counter"
2) "hello"
127.0.0.1:6379> mget hello counter
1) "world"
2) "1"
```

**通过py脚本清理所有key**

方法一：
```python
root@iZ947mgy3c5Z:/tmp# cat rds_3.py
import redis
client = redis.StrictRedis(host="127.0.0.1", port=6379)

keys_result = client.keys()

def clean_all_keys(client, keys):
    pipeline = client.pipeline(transaction=False)
    for key in keys_result:
        pipeline.delete(key)
    result = pipeline.execute()
    return result
res = clean_all_keys(client, keys_result)
print("Pipeline Clean Result: {}".format(res))
```

方法二：
```python
root@iZ947mgy3c5Z:/tmp# cat rds_4.py 
import redis
client = redis.StrictRedis(host="127.0.0.1", port=6379)

res = client.flushall()
print("Pipeline Clean Result: {}".format(res))
```

## 调用lua脚本
三个重要的函数
* eval(string Script, int KeyCount, string ... Params)
* evalsha(string Script)
* script_load(string sha, int KeyCount, string ... Params)


**eval()**
脚本内容
```root@iZ947mgy3c5Z:/tmp# cat rds_5.py 
import redis
client = redis.StrictRedis(host="127.0.0.1", port=6379)
script = "return redis.call('get', KEYS[1])"
print(client.eval(script, 1, "hello"))
```

执行结果

```
root@iZ947mgy3c5Z:/tmp# python3.5 rds_5.py 
b'world'
```

**script_load()、evalsha()**
脚本内容
```
import redis
client = redis.StrictRedis(host="127.0.0.1", port=6379)

script = "return redis.call('get', KEYS[1])"

script_sha = client.script_load(script)
res = client.evalsha(script_sha, 1, "hello")
print(res)
```

执行结果
```
root@iZ947mgy3c5Z:/tmp# python3.5 rds_6.py
b'world'
```
