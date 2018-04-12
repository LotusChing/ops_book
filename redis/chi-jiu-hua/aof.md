# AOF

默认redis持久化方案是RDB，而aof是没有开启，所以需要手动启用配置，好在redis支持动态配置

## 配置启用持久化

**方法一：动态开启AOF持久化**
```
root@iZ947mgy3c5Z:/opt/redis# redis-cli 
127.0.0.1:6379> config get appendonly 
1) "appendonly"
2) "no"
127.0.0.1:6379> config set appendonly yes
OK
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set name da
OK
127.0.0.1:6379> 
root@iZ947mgy3c5Z:/opt/redis# more data/appendonly.aof 
*2
$6
SELECT
$1
0
*3
$3
set
$4
name
$2
da
```

讲道理看到这我笑出声来了，咋这么眼熟，这不是就是Redis通信协议中RESP标准序列化后的命令么

**方法二：修改配置文件appendonly no改为yes**


## AOF持久化工作流程

简单记下流程
1. redis将RESP命令序列写入到aof_buf缓冲区中
2. AOF缓冲区根据策略将缓冲区的数据追加到$dir/$appendfilename配置命名的文件中
3. Redis会通过重写AOF文件以打到压缩的目的
4. Redis启动时会读取AOF实现数据的恢复

### 命令写入
#### AOF写入的格式是什么？ 为什么使用这种格式？ 优点是什么？ 缺点是什么？ 

通过上面也可以看到，文件中写入的是RESP文本格式数据，优点的话可能有一下两点
1. 这么做的有点是为了避免RDBchijiuhua中存在的兼容性问题
2. 其次文本的可读性和操作性较好可能也是设计的初衷之一


缺点的在于**文本格式的数据文件体积较大**，**恢复时载入较慢**，尤其是相较于RDB二进制数据文件而言


#### 为什么要先写到aof_buf缓冲区中？ 解决了什么问题？ 带来了什么问题？
如果没有aof_buf缓冲区，也就是每次发生修改操作都直接写入到文件系统中，而且redis是单线程的，那不意味着redis的性能完全受限于硬盘的性能了

所以为了避免这个问题，Redis使用aof_buf缓冲区作为内存到磁盘的一个过渡，然后根据一定的刷盘策略，进行数据持久化

使用aof_buf缓冲区虽然解决了性能的问题，但是也带来了一些问题，就是在极端情况下不同的刷盘策略的性能和数据安全性也是不同的


### 文件同步策略(刷盘策略)
aof_buf缓冲区的刷盘策略对应的配置参数是`appendfsync`，一共有三种策略
* always: 每次发生修改操作都直接通过系统调用fsync进行刷盘，最安全，性能影响最大，因为fsync的期间是阻塞的，**不建议配置，因为和redis的特定背道而驰**
* everysec: 将数据写入到aof_buf后，通过write系统调用操作，write调用会触发延迟写的机制，此时数据写到了系统的页缓冲中，Linux内核中通过页缓冲来提高磁盘IO的性能，具体页缓冲什么时候刷盘取决于特定的策略，例如达到特定的大小或者间隔，aof_buf的fsync是由操作系统专门的线程进程刷盘的，**推荐配置也是默认配置**
* no: 由操作系统进行刷盘，一般情况下最长30秒，由于时间间隔太长数据安全性无法保证，所以**通常不建议配置，除非不关心数据安全性**

### 文件重写

#### 什么是文件重写？ 文件重写的好处有哪些？ 
文件重写简单来说就是，将内存中的数据重新生成一份RESP写入命令，例如一个简单的k:v数据，name:da，他的写入命令就是set name da，RESP格式的呢就是，`*3\r\n$3r\nset\r\n$4\r\nname\r\n$2\r\nda\r\n`

文件重写有两个好处
1. 由于是直接基于当前内存中的最新数据，所以之前数据文件中的无效数据，例如del key2 hdel key2都不会存在，而例如lpush key3 value1 lpush key3 value2 都会被压缩为一条命令，lpush key3 value1 value2，所以重写后的体积会比之前有所减少，**PS: list、hash、set、zset合并时以64个元素为界**
2. 由于重写后的数据文件体积变小，Redis启动恢复数据时要载入的写入命令也就减少了，从而启动载入数据的速度更快了

#### 如何触发文件重写？
文件重写分为两种触发方式
**手动触发**

重写前的aof数据文件，可以看到name键被我修改过，由da->yo
```
root@iZ947mgy3c5Z:/opt/redis# cat data/appendonly.aof 
*2
$6
SELECT
$1
0
*3
$3
SET
$4
name
$2
da
*2
$6
SELECT
$1
0
*3
$3
set
$4
name
$2
yo*2
$6
SELECT
$1
0
*3
$3
SET
$4
name
$2
yo

```

执行`bgrewriteaof`命令手动触发文件重写
```
127.0.0.1:6379> bgrewriteaof
Background append only file rewriting started
```

重写后的数据文件，可以看到Redis直接基于当前最新的数据生成的写入命令
```
root@iZ947mgy3c5Z:/opt/redis# cat data/appendonly.aof 
*2
$6
SELECT
$1
0
*3
$3
SET
$4
name
$2
yo
```

** 自动触发 **
自动触发AOF文件重写有两个关键的条件参数
* auto-aof-rewrite-min-size: 当aof数据文件最小达到该值时
* auto-aof-rewrite-percentage: 当前aof空间和上次aof重写后的空间的比值



$$自动触发 = aof_current_size > auto-aof-rewrite-min-size && (aof_current_size - aof_base_size) / aof_current_size >= auto-aof-rewrite-percentage$$


## AOF持久化内部流程
草图
![](/assets/bgrewriteaof_flow.jpg)


流程说明
1. 主进程接收到bgrewrite指令
2. 主进程fork子进程，fork期间阻塞，fork成功由子进程执行重写，主进程接收新命令
3.1 为了保证数据的完整性，新的写入命令会写到aof_buf中
3.2 由于子进程只能获取到父进程fork时的数据，如果仅基于那时的数据进行重写，肯定会丢失重写期间的写入命令，所以新的写入命令除了写到aof_buf，还会写到aof_rewrite_buf
4. 子进程基于fork时的内存数据进行重写
5. 子进程通知父进程已经将数据以RESP重写到新的aof文件中，更新info中aof_*信息
6. 父进程将重写期间的写入命令追加到新的aof文件
7. 使用新aof替换老的aof文件，整个bgrewriteaof完成



## AOF校验和修复
校验
```
root@iZ947mgy3c5Z:/opt/redis# redis-check-aof data/appendonly.aof 
AOF analyzed: size=54, ok_up_to=54, diff=0
AOF is valid
```

修复
```
root@iZ947mgy3c5Z:/opt/redis# redis-check-aof --fix data/appendonly.aof 
AOF analyzed: size=99, ok_up_to=77, diff=22
This will shrink the AOF from 99 bytes, with 22 bytes, to 77 bytes
Continue? [y/N]: y
Successfully truncated AOF
```

手动破坏了aof文件里的值以测试修复功能，感觉没那么只能，并不像我以为的那样，只truncated掉有问题的部分，例如：a、b、c，手动破坏了b键，实际影响可能是abc三个键，而且最后恢复回来的值也可能是错误的