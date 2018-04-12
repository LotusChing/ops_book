# 常见问题

## 实验中遇到的异常问题
### 问题1：manager的ip更换导致worker节点异常

#### 异常现象
1. docker server ls全都处于0/n，显然都处于非健康状态
2. docker server ps server_name，虽然DESIRED STATE都处于Running状态，但是CURRENT STATE列显示Pending好久了
3. docker node ls，发现节点Status状态列处于Down，说明Worker已经断开连接了

#### 问题定位
首先检查了基础网络连通性，发现都是OK的~，然后Google下没发现什么有用的讯息，最后不经意想起worker节点加入时填写的ip地址，那个ip地址是DHCP获取的，会不会租约到期了，所以IP变了呢
最后发现果然IP变了，于是将两个node重新加入了一遍，连接manager时用的是固定的IP，然后就恢复了~


### 问题2：manager和worker版本不一致
#### 问题现象

#### 问题定位
Google
1. 升级Docker
```docker
apt-get -y install apt-transport-https ca-certificates curl software-properties-common
add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
apt-get update
apt-get install docker-ce=17.12.0~ce-0~ubuntu
```

2. 重建Swarm即可