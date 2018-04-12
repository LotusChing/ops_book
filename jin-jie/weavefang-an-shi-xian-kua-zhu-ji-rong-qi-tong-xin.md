# Weave方案实现跨主机容器通信

由于macvlan无法满足需求，所以看下Weave方案

## 环境说明

* 系统：Ubuntu 14.04.1 LTS
* 内核：3.13.0-32-generic
* Docker：17.12.0-ce

## 节点说明
### node-50
* eth0：192.168.2.50(仅主机)
* eth1：192.168.0.177(桥接)

### node-60
* eth0：192.168.2.60(仅主机)
* eth1：192.168.0.178(桥接)


## 安装

1. 下载weave
```shell
# curl -L git.io/weave -o /usr/local/bin/weave
# chmod +x /usr/local/bin/weave
```

## 创建网络
**1. 初始化weave网络环境**

**node-50**
第一个启动的节点只需要执行下面命令就可以了，执行后会自动下载weave相关镜像，并创建weave网络环境，并且监听0.0.0.0:6783，这是节点间通信的端口
```shell
# weave launch
```

**node-60**
```shell
# weave launch 192.168.0.178
```


**2. 检查weave网络情况**
```shell
# weave status 
# weave status connections
<- 192.168.0.177:39037   established fastdp 2a:37:1c:8f:e8:cf(node-50) mtu=1376
```

## 创建容器
**node-50**
```shell
docker run --name a1 -ti weaveworks/ubuntu
```

**node-60**
```shell
docker run --name a2 -ti weaveworks/ubuntu
```


## 测试连通性
**node-50**
```shell
root@a1:/# ping -c1 -W1 a2 &>/dev/null && echo "OK" || echo "No"
OK
root@a1:/# ping -c1 -W1 192.168.0.1 &>/dev/null && echo "OK" || echo "No"
OK
root@a1:/# ping -c1 -W1 www.baidu.com &>/dev/null && echo "OK" || echo "No"
OK
root@a1:/# ping -c1 -W1 a2.weave.local &>/dev/null && echo "OK" || echo "No"
OK
```


**node-60**
```shell
root@a2:/# ping -c1 -W1 a1 &>/dev/null && echo "OK" || echo "No"
OK
root@a2:/# ping -c1 -W1 192.168.0.1 &>/dev/null && echo "OK" || echo "No"
OK
root@a2:/# ping -c1 -W1 www.baidu.com &>/dev/null && echo "OK" || echo "No"
OK
root@a2:/# ping -c1 -W1 a1.weave.local &>/dev/null && echo "OK" || echo "No"
OK
```

## 新加节点

测试新加节点会自动同步数据，而且三个节点中所有容器是可以相互通信的

## 总结

基本需求都可以实现，而且简单好用！