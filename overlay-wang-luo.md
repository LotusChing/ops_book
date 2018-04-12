# Overlay网络

Swarm中使用Overlay超级方便，不需要借助Consul等第三方存储服务来存储网络配置信息，Swarm内置类似的服务实现了

## 创建、应用、测试Overlay网络
```docker
root@node-40:~# docker network create -d overlay overlay_net
root@node-40:~# docker network ls -f "DRIVER=overlay"
NETWORK ID          NAME                DRIVER              SCOPE
nun25wf6hnpv        ingress             overlay             swarm
qa593lu08jbj        overlay_net         overlay             swarm
```

1. 创建Service并应用Overlay网络
```docker
root@node-40:~# docker service create --replicas 2 --network overlay_net --name ovl_my_web nginx:1.12
```

2. 检查Service创建启动情况
```docker
root@node-40:~# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
hy489wd5d6a7        ovl_my_web          replicated          2/2                 nginx:1.12          
```

3. 此时Manager节点通知让Worker节点创建容器前，会先将overlay网络也同步过去
```docker
root@node-50:~# docker network ls -f "DRIVER=overlay"
NETWORK ID          NAME                DRIVER              SCOPE
qa593lu08jbj        overlay_net         overlay             swarm
root@node-60:~# docker network ls -f "DRIVER=overlay"
NETWORK ID          NAME                DRIVER              SCOPE
qa593lu08jbj        overlay_net         overlay             swarm
```

4. 检查网络情况，由于nginx默认镜像中没有ping命令，所以更新下源，然后安装一下
```docker
root@node-50:~# docker exec -it ovl_my_web.2.7v624xu7otuh0zr0xd9rcmh8p bash
root@03a2bd124965:/# apt-get update
root@03a2bd124965:/# apt-get install iputils-ping
```

5. 测试网络情况
另外一台主机的容器: OK~
```docker
root@03a2bd124965:/etc/apt# ping -c1 -W1 ovl_my_web.1.kfm7cftwguhhqrso4blxhkzcq 
PING ovl_my_web.1.kfm7cftwguhhqrso4blxhkzcq (10.0.0.3) 56(84) bytes of data.
64 bytes from ovl_my_web.1.kfm7cftwguhhqrso4blxhkzcq.overlay_net (10.0.0.3): icmp_seq=1 ttl=64 time=1.12 ms
```

外网出口网关: OK~
```docker
root@03a2bd124965:/etc/apt# ping -c1 -W1 192.168.0.1                           
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_seq=1 ttl=63 time=2.23 ms
```


测试公网访问: OK~
```docker
root@03a2bd124965:/etc/apt# ping -c1 -W1 www.baidu.com
PING www.a.shifen.com (220.181.112.244) 56(84) bytes of data.
64 bytes from 220.181.112.244 (220.181.112.244): icmp_seq=1 ttl=52 time=7.99 ms
```


OK，都是成功访问的，另外一台主机一样~


## 将现有的Service加入Overlay网络

此时Docker Swarm会删除重建容器来应用网络
```docker
root@node-40:~# docker service update --network-add overlay_net my_web
root@node-40:~# docker service ps -f "DESIRED-STATE=Running" my_web
```


重建后连接容器，检查网络情况
```docker
root@node-50:~# docker exec -it my_web.2.1ra3aua73egpw21rm3vuygm5t bash
root@fed217774102:/# ping ovl_my_web.1.kfm7cftwguhhqrso4blxhkzcq
PING ovl_my_web.1.kfm7cftwguhhqrso4blxhkzcq (10.0.0.3) 56(84) bytes of data.
64 bytes from ovl_my_web.1.kfm7cftwguhhqrso4blxhkzcq.overlay_net (10.0.0.3): icmp_seq=1 ttl=64 time=0.752 ms
root@fed217774102:/# ping my_web.5.ptrbadb8zvazekkr6bt99kwua
PING my_web.5.ptrbadb8zvazekkr6bt99kwua (10.0.0.7) 56(84) bytes of data.
64 bytes from my_web.5.ptrbadb8zvazekkr6bt99kwua.overlay_net (10.0.0.7): icmp_seq=1 ttl=64 time=1.00 ms
root@fed217774102:/# ping www.baidu.com                     
PING www.a.shifen.com (180.149.132.151) 56(84) bytes of data.
64 bytes from 180.149.132.151 (180.149.132.151): icmp_seq=1 ttl=53 time=6.82 ms
```



## 将Service移除Overlay网络
移除Service网络和添加一样，都是删除重建容器完成的
```docker
root@node-40:~# docker service update --network-rm overlay_net my_web
root@node-40:~# docker service ps -f "DESIRED-STATE=Running" my_web
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                    ERROR               PORTS
mqp6a74k8mei        my_web.1            nginx:1.12          node-60             Running             Running 7 seconds ago                                
h72sj4v3d86f        my_web.2            nginx:1.12          node-50             Running             Running less than a second ago                       
j9x0gmll53dt        my_web.3            nginx:1.12          node-60             Running             Running less than a second ago                       
kv4ouucrgbf5        my_web.4            nginx:1.12          node-60             Running             Running less than a second ago                       
m2e881khny4n        my_web.5            nginx:1.12          node-50             Running             Running 6 seconds ago                                
whg9nwv4icux        my_web.6            nginx:1.12          node-50             Running             Running less than a second ago                  
```
