# Grains

## Grains是什么？

minion在启动时会收集一些minion本地的数据，这部分数据是是一般不会发生变化的数据，例如操作系统版本，内核版本，CPU信息，内存信息等等，这部分数据只有重启minion才会刷新，所以称之为静态数据。

## Grains有什么用？

1. 为CMDB提供资产数据支持
2. 为目标选择提供数据支持，例如为指定版本的操作系统更新某个包
3. 配置管理

## 如何获取Grains中的数据？

1. 获取所有的grains keys，`salt 'server' grains.ls`
2. 获取所有的grains items(K:V)，`salt 'server' grains.items`
3. 获取指定的grains k:v
 ```
 root@server:~# salt 'server' grains.item os
server:
    ----------
    os:
        Ubuntu
 ```
 
## 如何将Grains中的数据作为目标条件选择？
使用`-G`参数，例如找到系统为Ubuntu的minion上执行`free -m`
```
root@server:~# salt -G 'os:Ubuntu' cmd.run 'free -m'
server:
                 total       used       free     shared    buffers     cached
    Mem:         16004      15636        367         13        201       6431
    -/+ buffers/cache:       9004       6999
    Swap:        16336       2943      13393
```

## 怎么自定义添加Grains数据？
修改/etc/salt/minion
```
root@server:~# vim /etc/salt/minion
120 grains:
121   roles:
122     - test
root@server:~# /etc/init.d/salt-minion restart
``` 

重启后，尝试使用自定义的grains数据作为条件
```
root@server:~# salt -G 'roles:test' cmd.run 'free -m'
server:
                 total       used       free     shared    buffers     cached
    Mem:         16004      15636        368         13        201       6431
    -/+ buffers/cache:       9003       7000
    Swap:        16336       2943      13393
```

## 如何将Grains数据从配置文件从分离出来？
新建文件cat /etc/salt/grains，然后把数据定义写到这里面，minion会自动到这个文件里找
```
root@server:~# cat /etc/salt/grains 
myname:
    - LotusChing
```

这里有个小技巧，可以不重启热载入grains配置(无论是写到grains、minion都生效)
```
root@server:~# salt 'server' saltutil.sync_grains
```

测试调用
```
root@server:~# salt -G 'myname:LotusChing' cmd.run 'free -m'
server:
                 total       used       free     shared    buffers     cached
    Mem:         16004      15685        318         13        201       6431
    -/+ buffers/cache:       9052       6951
    Swap:        16336       2941      13395
```

## 如何将Grains应用到状态文件中？
配置如下：
```
root@server:/srv/salt# cat /srv/salt/top.sls 
base:
  'myname:LotusChing':
    - match: grain
    - initial.basic
```
myname(k):LotusChing(v)
match代码使用grains进行匹配

执行测试
```
root@server:/srv/salt# salt '*' state.highstate
server:
----------
          ID: basic-install
    Function: pkg.installed
        Name: lrzsz
      Result: True
     Comment: Package lrzsz is already installed
     Started: 14:41:42.624433
    Duration: 633.35 ms
     Changes:   
----------
          ID: basic-install
    Function: pkg.installed
        Name: htop
      Result: True
     Comment: Package htop is already installed
     Started: 14:41:43.257928
    Duration: 4.686 ms
     Changes:   

Summary for server
------------
Succeeded: 2
Failed:    0
------------
Total states run:     2
Total run time: 638.036 ms
```

## 如何动态生成grains？

说起来可能会比较抽象，但是举个实际的例子，所有服务器默认是没有角色的，但是当我们通过cmdb上给服务器配置了个角色，以便于后续管理时，我们会在在数据库中定义好服务器角色，以便于使用，但是salt-master没法和数据库进行数据交互啊，所以这时就需要在salt-master这里动态生成grains，然后推到指定的服务器上

如何实现呢？


```
# mkdir /srv/salt/_grains
# cat /srv/salt/_grains/my_grains.py
#!/usr/bin/env python
#coding:utf8

def my_grains():
    grains = {'tag': 'Yo'}
    return grains
# root@server:/srv/salt/_grains# salt '*' saltutil.sync_grains
server:
    - grains.my_grains
# root@server:/srv/salt/_grains# salt '*' grains.item tag
server:
    ----------
    tag:
        Yo
```
搞定，通过在salt-master的`_grains`目录开发`my_grains.py`脚本实现动态添加到minion中的grains里，使用`saltutil.sync_grains`推送配置时，`my_grains.py`也是会被推到minion端的`/var/cache/minion/files`下

其实这个有几个不太妥善的小问题
1. py文件名应该为minion_id，这样方便标识
2. 函数名和内容也需要根据需求进行替换


## 各种姿势的grains定义优先级是？
1. 原生
2. /etc/salt/grains
3. /etc/salt/minion
4. py

