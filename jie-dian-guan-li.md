# 节点管理

## 列出集群节点信息
```
root@node-50:~# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
tim72sbpcd33765m3kg00oghk     node-40             Ready               Active              Reachable
js01jwztashf7ix2l536f3e25 *   node-50             Ready               Active              Leader
x1jg9nuoiw7olrhkymxd9ivve     node-60             Ready               Active              Reachable
```


## 列出指定节点详细信息
默认是类似Json格式返回，`--pretty`更易于阅读一些
```docker
root@node-50:~# docker node inspect node-40 --pretty
ID:			tim72sbpcd33765m3kg00oghk
Hostname:              	node-40
Joined at:             	2018-02-26 03:09:39.732577983 +0000 utc
Status:
 State:			Ready
 Availability:         	Active
 Address:		192.168.2.40
Manager Status:
 Address:		192.168.2.40:2377
 Raft Status:		Reachable
 Leader:		No
Platform:
 Operating System:	linux
 Architecture:		x86_64
 ...
```


## 提权指定节点
woker ->  manager
```
root@node-50:~# docker node promote node-40
```

## 降权指定节点
```docker
root@node-50:~# docker node demote node-60
Manager node-60 demoted in the swarm.
root@node-50:~# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
tim72sbpcd33765m3kg00oghk     node-40             Ready               Active              Reachable
js01jwztashf7ix2l536f3e25 *   node-50             Ready               Active              Leader
x1jg9nuoiw7olrhkymxd9ivve     node-60             Ready               Active              
```

常用的就这几个，其他的根据`--help`帮助使用即可~