# Swarm 开始

Swarm是Docker默认的容器集群管理方案，所以上手简单，学习成本很低

## 环境说明

* 系统：Ubuntu 14.04.1 LTS
* 内核：3.13.0-32-generic
* Docker：17.12.0-ce

## 节点说明
### node-40
* eth0：192.168.2.50(仅主机)
* eth1：192.168.0.179(桥接)

### node-50
* eth0：192.168.2.50(仅主机)
* eth1：192.168.0.177(桥接)

### node-60
* eth0：192.168.2.60(仅主机)
* eth1：192.168.0.178(桥接)



## 快速开始
首先在40上创建Swarm集群，创建集群的一方默认就是manager角色，swarm共有两种角色

* manager：负责Swarm集群的管理，接收docker客户端的指令，然后调度分配Service的创建任务到worker节点上
* worker：负责接收manager的指令创建具体的容器

### 创建集群
`--advertise-addr`参数是manager的外部地址，这个地址最好是固定的，不然会出现问题

```docker
docker swarm init --advertise-addr 192.168.0.179
```

### 加入集群
创建完Swarm集群后，会返回用以其他node加入的命令，这个命令是以worker角色加入的
```docker
docker swarm join --token SWMTKN-1-6czw4is8t2lhg76uud7emybbj8wa15ulcucybpq102d82v2dg9-ak45pt0l44tb3gbs2e0idjqat 192.168.0.179:2377
```

如果需要以manager加入，则需要在manager节点执行一下命令，就会返回以manager角色加入的命令
```docker
root@node-40:~# docker swarm join-token manager 
```


### 检查集群节点
通过`docker node ls`查看集群节点状态，各节点STATUS为Ready即为ok~
