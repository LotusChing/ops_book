# 容器网络

容器间通信最开始是通过其内部IP地址，但默认容器内部IP重启后会变化，所以后来使用的是--link来连接容器，这样当被连接的容器哪怕重启后IP变化依旧可以访问到对方，其原理就是hosts里动态变化的主机名解析，当被连接对象的IP变化时，里呢连接一方的hosts也会变化。



但是当看到《Docker入门到实践》中的内容后，发现同网络内的容器，任何一个容器都可以不用link，实现可以通过主机名实现访问对方容器，而且效果和link一样，简直了！


## 创建容器网络
```docker
# docker network create -d bridge flask_net# docker network create -d bridge flask_net
ba2b805768d58d42888d3ea9666273fd2450d0f88da41dd929f21755c66a1890
```

## 列出所有Docker网络
```docker
# docker network ls
NETWORK ID          NAME                DRIVER
48f36c36f9ee        bridge              bridge              
ba2b805768d5        flask_net           bridge              
b21015a35de4        host                host                
2d074c7ea467        none                null                
78871f821087        wecan               bridge              
```

## 创建容器并加入自定义网络
```docker
# docker run -d --name flask_pro1 --net flask_net flask_project_1:latest 
3638938d7c01cc62e5b9c7c4ee445e73e998ab7b023b3e3ad1931f00599b1132
# docker run -d --name flask_pro2 --net flask_net flask_project_1:latest 
55f3a94c1bb8856f4ee9508b10ca3f4e2116dd55ca00073c15b5315999eba93a
```

测试访问
```
# docker exec -it flask_pro2 ping -c1 flask1
PING flask1 (192.168.0.2): 56 data bytes
64 bytes from 192.168.0.2: icmp_seq=0 ttl=64 time=0.147 ms
```
