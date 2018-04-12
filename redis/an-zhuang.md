# 安装Redis

安装redis步骤比较简单

**1. 下载源码包**
最新的Redis稳定版已经是4.0.6，但《Redis开发和运维》书中使用的是3.0.7，为了避免学习时出现奇奇怪怪的问题，我还是先老实点用3.0.7，新版本后续再折腾吧

```
cd /usr/local/src/
wget http://download.redis.io/releases/redis-3.0.7.tar.gz
```

**2. 解压/软链**
软链接是为了后续方便升级，这是个习惯
```

ln -s /opt/redis-3.0.7 /opt/redis
```


**3. 编译安装**
编译前确保安装了GCC等编译时需要的工具
```
cd /opt/redis
make && make install
```

**4. 测试**
redis的二进制文件会自动编译到/usr/local/bin/下，所以正常情况下，可以直接使用`redis-cli`命令来查看版本号
```
[root@bj-vmware-test1 redis]# redis-cli -v
redis-cli 3.0.7
```

**5. 补充说明**
* redis-server: redis server启动文件
* redis-cli: redis命令行客户端
* redis-benchmark: redis基准测试工具
* redis-check-aof: redis AOF持久化文件检测和修复工具
* redis-check-dump: redis RDB持久化文件检测和修复工具
* redis-sentinel: redis-sentinel(好像是集群方案)


