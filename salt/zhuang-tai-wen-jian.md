# 状态文件

Salt的状态文件使用的YAML语法，YAML语法比较简单，下面是几个主要的点

## YAML语法小贴士
1. 使用单引号''
2. 注意层级缩进，两个空格为单位
3. key: value，中间有个空格
4. 短横线 - 代表为列表中的元素

## 状态文件在哪里写？
状态文件的路径定义在/etc/salt/master里

Salt可以针对不同的环境单独设置状态文件路径，默认Base配置好了一个路径，不过是注释掉的，所以取消注释就可以了，如果文件系统没有这个路径，咱手动创建下
```
# vim /etc/salt/master
 599 file_roots:
 600   base:
 601     - /srv/salt/
```

重启salt-master载入新配置
```
# /etc/init.d/salt-master restart
```

## 状态文件怎么写？
这里贴出一个简单的小例子，initial和web目录是为了区分状态文件的类型的，这样方便我们分类管理
```yaml
root@server:/srv/salt# ls
initial  top.sls  web
root@server:/srv/salt# cat initial/basic.sls 
basic-install:   # 任务的名字
  pkg.installed: # 调用pkg模块的installed方法
    - names:     # names是个列表，里面的元素是要安装的包的名字
      - lrzsz    # 上传下载文件的包
      - htop     # 查看系统资源的包
```


## 如何执行写好的状态文件？
通过执行state.sls执行指定的sls文件，例如：`root@server:/srv/salt# salt 'server' state.sls initial.basic`，initial是包，basic是模块，所以连起来就是，使用state模块的sls方法，执行initial包里的basic模块。

### 小例子
下面是执行结果：
```shell
root@server:/srv/salt# salt 'server' state.sls initial.basic
server:
----------
          ID: basic-install
    Function: pkg.installed
        Name: lrzsz
      Result: True
     Comment: Package lrzsz is already installed
     Started: 10:44:02.410871
    Duration: 639.356 ms
     Changes:   
----------
          ID: basic-install
    Function: pkg.installed
        Name: htop
      Result: True
     Comment: Package htop is already installed
     Started: 10:44:03.050360
    Duration: 4.807 ms
     Changes:   

Summary for server
------------
Succeeded: 2
Failed:    0
------------
Total states run:     2
Total run time: 644.163 ms
```
执行结果里包含了很多东西，例如ID(我理解为任务标识)，用到了什么执行函数，执行多长时间，是否正常完成执行等等，这些东西估计后续做运维平台开发时都会用到。

### 状态文件到底如何执行的？
我们不能光知其然还要知其所以然，到底咱在敲下这个执行状态的命令后，到底发生了什么，简单思考下

1. 首先salt确定了执行目标的范围，嗯，就一个server
2. 接着salt调用了state模块中sls方法
3. 由于状态文件是在master端编写的，minion端并没有，所以master要把状态模块发送给了minion端，存放路径在
  ```
  root@server:/srv/salt# ls   /var/cache/salt/minion/files/base/initial/
basic.sls
  ```
4. minion开始执行状态模块，返回执行结果


## TOP file是什么？

先不说topfile是什么，先说一个需求，假如我们写了很多模块，例如初始化操作系统的，安装基础包的，安装web服务的，安装db服务等等，但是假如咱用state.sls就需要手动挨个执行状态文件，几个还好，假如是多个那就很忧伤了，如果有一个地方能够定义服务器和状态文件的对应关系就好了，TOP file就是干这个的

topfile是个名词，一般都用top.sls来命名，使用topfile功能需要修改配置文件，其实取消下注释就行了
```
root@server:/srv/salt# vim /etc/salt/master
 517 state_top: top.sls
```

## topfile 在那写？

这个需要说明一点就是，top.sls需要定义在base环境所定义的路径下，结合上面base环境的定义，我们应该吧top.sls，写在/srv/salt/top.sls这里


## topfile 怎么写？
这是个小例子，这里使用到了刚才写的状态模块
```
root@server:/srv/salt# cat top.sls 
base:
  'server':
    - initial.basic
```
* base代表的是环境
* server是minion_id，也就是服务器标识
* -关联的状态模块，关联多个的话，就新起一行同样的格式


## topfile 如何执行？
使用state.highstate模块方法执行
```
root@server:/srv/salt# salt 'server' state.highstate
```

## topfile 执行应该注意什么？
1. 使用`salt '*' state.show_top`，确认minion所要执行的状态文件
2. 使用`salt 'server' state.highstate test=True`先测试，salt会返回它要执行什么，咱最好确认下，在执行`salt 'server' state.highstate`
3. 尽量不要使用 *
4. 既然用了，就用到底，例如实际集群节点数和状态文件不一致，当执行topfile导致部分节点失效，从而宕机，这个案例来自于知名外卖网站
