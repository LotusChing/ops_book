# 执行模块

Salt内置很多模块，cmd.run是常用的一种，不过还有其他很多的模块也非常好用，不一一记了，[Salt远程执行模块](https://www.unixhot.com/docs/saltstack/ref/modules/all/index.html#all-salt-modules "Salt远程执行模块")


这些模块也存在文件系统中，路径是这里`/usr/lib/python2.7/dist-packages/salt/modules`，可以看到一共450多个模块
```
root@server:/usr/lib/python2.7/dist-packages/salt/modules# ls *.py |wc -l
455
```

## network

1. 获取minion tcp连接
 ```
 salt 'server' network.active_tcp
 ```

2. 获取minion arp表
 ```
 salt 'server' network.arp
 ```


## service

1. 当前服务是否在运行
 ```
 salt 'server' service.available ssh
 ``` 
2. 查看当前所有运行的服务
 ```
 salt 'server' service.get_all
 ```
 
3. 启动、停止、重启、重载服务
 ```
 salt 'server' service.start ssh
 salt 'server' service.stop ssh #别敲
 salt 'server' service.restart ssh
 salt 'server' service.reload ssh
 ```

## cp
1. master拷贝文件到minion
 ```
 salt-cp '*' /etc/hostname /tmp/haha
 ```

2. 拷贝文件夹
 ```
 salt-cp '*' testdir /tmp/
 ```
 

