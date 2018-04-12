# 管理节点高可用

## manager高可用

原理和Redis Sentinel一样都是，Raft协议投票选举

1. 首先将两个worker节点提权
```docker
root@node-40:~# docker node promote node-50 
Node node-50 promoted to a manager in the swarm.
root@node-40:~# docker node promote node-60 
Node node-60 promoted to a manager in the swarm.
```

2. 提权后的节点可以执行manager运行的命令，例如查看Service、创建Service等
```docker
root@node-50:~# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
tim72sbpcd33765m3kg00oghk     node-40             Ready               Active              Leader
js01jwztashf7ix2l536f3e25 *   node-50             Ready               Active              Reachable
x1jg9nuoiw7olrhkymxd9ivve     node-60             Ready               Active              Reachable
```

3. 测试停掉现在的manager
```docker
root@node-40:~# service docker  stop
docker stop/waiting
```

4. 观察现象

可以看40原manager状态发送了变化，50已经晋升为Leader
```
root@node-50:~# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
tim72sbpcd33765m3kg00oghk     node-40             Unknown             Active              Unreachable
js01jwztashf7ix2l536f3e25 *   node-50             Ready               Active              Leader
x1jg9nuoiw7olrhkymxd9ivve     node-60             Ready               Active              Reachable
```

再次启动40的docker服务，发现又恢复了
```docker
root@node-40:~# service docker  start
docker start/running, process 34005
root@node-40:~# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
tim72sbpcd33765m3kg00oghk *   node-40             Ready               Active              Reachable
js01jwztashf7ix2l536f3e25     node-50             Ready               Active              Leader
x1jg9nuoiw7olrhkymxd9ivve     node-60             Ready               Active              Reachable
```

重要的是，manager从挂掉到投票选举完成，这部分时间里Service是不受影响的，唯一受影响的就是Manager相关的操作，例如查看、创建Service等
