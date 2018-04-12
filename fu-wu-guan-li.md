# 服务管理

## 服务和任务
* 任务 （Task）是 Swarm 中的最小的调度单位，目前来说就是一个单一的容器。

* 服务 （Services） 是指一组任务的集合，服务定义了任务的属性。服务有两种模式：
  * replicated services 按照一定规则在各个工作节点上运行指定个数的任务
  * global services 每个工作节点上运行一个任务
* 两种模式通过 docker service create 的 --mode 参数指定。

## 创建服务
```docker
docker service create --replicas 2 --name lotus busybox tail -F /var/log/dmesg
```

## 列出/过滤服务
列出服务
```docker
root@node-40:~# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
iwnj041mw0xn        hello               replicated          2/2                 busybox:latest      
n7e2zj2f8i9r        lotus               replicated          2/2                 busybox:latest      
```


如果Service过多可以通过`-f`指定参数以过滤服务
```docker
root@node-40:~# docker service ls -f "name=lotus"
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
n7e2zj2f8i9r        lotus               replicated          2/2                 busybox:latest      
```

## 列出/过滤任务(Task)
之前说道了一个Service会包含一个或多个容器，可以通过`ps`参数查看容器的状态，这里的容器称之为Task
```docker
root@node-40:~# docker service ps lotus
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
wqi3c5gs6q48        lotus.1             busybox:latest      node-50             Running             Running 3 minutes ago                       
6sxidbxzgkwu        lotus.2             busybox:latest      node-60             Running             Running 3 minutes ago                       
```


过滤Task
```docker
root@node-40:~# docker service ps -f "node=node-50" lotus
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
wqi3c5gs6q48        lotus.1             busybox:latest      node-50             Running             Running 3 minutes ago                       
```


## 扩容/缩容服务
通过scale指令可以很方便的一个Service进行扩容或者缩容

```docker
root@node-40:~# docker service scale lotus=3
lotus scaled to 3
root@node-40:~# docker service ps lotus
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
wqi3c5gs6q48        lotus.1             busybox:latest      node-50             Running             Running 4 minutes ago                       
6sxidbxzgkwu        lotus.2             busybox:latest      node-60             Running             Running 4 minutes ago                       
5lgzysyxjp4a        lotus.3             busybox:latest      node-50             Running             Running 3 seconds ago                       
```

## 获取服务详细信息
第一种返回Json，第二种便于阅读
```docker
root@node-40:~# docker service inspect hello
root@node-40:~# docker service inspect hello --pretty
```

## 滚动/间隔更新
```
root@node-40:~# docker service create --replicas 2 --name redis --update-delay 10s redis:3.0.6
root@node-40:~# docker service update --image redis:3.0.7 redis
```
更新策略可以通过`docker service inspect redis --pretty`查看
```docker
...
UpdateConfig:
 Parallelism:	1
 Delay:		10s
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
...
```


## 创建、更新、回滚
```
# docker service create --name my_web \
	--replicas 6 \
	--update-delay 10s \
	--update-parallelism 2 \
	--update-failure-action rollback \
	--rollback-parallelism 2 \
	--rollback-monitor 20s \
	nginx:1.12
# docker service ps my_web
```

更新，可以看到是并发2个执行更新的
```
root@node-40:~# docker service update --image nginx:1.13 my_web 
root@node-40:~# docker service ps my_web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE              ERROR               PORTS
q6j1s65l116c        my_web.1            nginx:1.13          node-60             Running             Preparing 10 seconds ago                       
kl5ae4w5boh1         \_ my_web.1        nginx:1.12          node-50             Shutdown            Shutdown 8 seconds ago                         
vneuky0ivupt        my_web.2            nginx:1.13          node-50             Running             Preparing 10 seconds ago                       
f6170j8f7wph         \_ my_web.2        nginx:1.12          node-60             Shutdown            Shutdown 8 seconds ago                         
saih08dad4x5        my_web.3            nginx:1.12          node-50             Running             Running 2 minutes ago                          
o0ted8mul45m        my_web.4            nginx:1.12          node-60             Running             Running 3 minutes ago                          
sj8vqi0bjfep        my_web.5            nginx:1.12          node-50             Running             Running 2 minutes ago                          
se16vb2lhgv7        my_web.6            nginx:1.12          node-60             Running             Running 3 minutes ago                          
```

测试更新到busybox，由于busybox启动后不会常驻前台，所以理论上会启动失败，看Swarm是否能够执行回滚
```
root@node-40:~# docker service update --image busybox my_web
root@node-40:~# docker service ps my_web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
ys01or2wc01u        my_web.1            nginx:1.12          node-60             Running             Running about a minute ago                       
yyb0x879wslq        my_web.2            nginx:1.12          node-60             Running             Running 8 seconds ago                            
f846l2w197yi         \_ my_web.2        busybox:latest      node-60             Shutdown            Complete 14 seconds ago                          
odyjcfw2rfi9         \_ my_web.2        busybox:latest      node-60             Shutdown            Complete 21 seconds ago                          
kbxszo8dxhzc         \_ my_web.2        nginx:1.12          node-50             Shutdown            Shutdown 23 seconds ago                          
ymqtmrnte4vg        my_web.3            nginx:1.12          node-50             Running             Running 14 seconds ago           
...
root@node-40:~# docker service ls -f "name=my_web"
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
n2g46nc5iaoy        my_web              replicated          6/6                 nginx:1.12          
```

可以看到最终还是回滚了，没有运行busybox~

在试下手动回滚，首先更新my_web
```
root@node-40:~# docker service update --image nginx:1.13 my_web 
dockmy_web
Since --detach=false was not specified, tasks will be updated in the background.
In a future release, --detach=false will become the default.
root@node-40:~# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
3n0ixuvnbgcg        redis               replicated          2/2                 redis:3.0.7         
iwnj041mw0xn        hello               replicated          2/2                 busybox:latest      
n2g46nc5iaoy        my_web              replicated          6/6                 nginx:1.13          
n7e2zj2f8i9r        lotus               replicated          3/3                 busybox:latest      
```

然后尝试回滚，可以看到手动回滚OK了，而且并发2个进行回滚
```
root@node-40:~# docker service update --rollback  my_web
my_web
Since --detach=false was not specified, tasks will be updated in the background.
In a future release, --detach=false will become the default.
root@node-40:~# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
3n0ixuvnbgcg        redis               replicated          2/2                 redis:3.0.7         
iwnj041mw0xn        hello               replicated          2/2                 busybox:latest      
n2g46nc5iaoy        my_web              replicated          4/6                 nginx:1.12          
n7e2zj2f8i9r        lotus               replicated          3/3                 busybox:latest      
```


## 查看服务日志
```docker
root@node-40:~# docker service logs my_web
```

## 删除服务
```docker
root@node-40:~#  docker service rm my_web
```