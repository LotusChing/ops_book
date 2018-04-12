# Macvlan方案实现跨主机容器通信

## 环境说明

* 系统：Ubuntu 14.04.1 LTS
* 内核：3.13.0-32-generic
* Docker：17.12.0-ce

## 节点说明
### node-50
* eth0：192.168.2.50(仅主机)
* eth1：192.168.0.175(桥接)

### node-60
* eth0：192.168.2.60(仅主机)
* eth1：192.168.0.187(桥接)

## 需求说明
1. 实现node50、60节点容器相互通信
2. 实现node50、60节点容器可以上外网

## 方案一：不创建子网卡实现
1. 首先在两台宿主机上配置将能够上公网的网卡开启混杂模式，不然无法实现最终效果
```shell
# ip link set eth1 promisc on
```

2. 创建网络，在两个节点分别执行
```docker
# docker network create -d macvlan --subnet=192.168.0.0/24 --gateway=192.168.0.1 -o parent=eth1 pub_net
```

3. 在node-50创建p_10容器，由于Docker默认不会检测IP是否存在，所以最好手动避免IP重复
```docker
# docker run -it --name p_10 --net=pub_net --ip=192.168.0.10 busybox sh
```

4. 在node-60上创建p_20容器，同样指定ip地址
```docker
# docker run -it --name p_20 --net=pub_net --ip=192.168.0.20 busybox sh
```

5. 测试访问跨节点容器和外部网络
**50节点**
```shell
/ # ping -c1 -W1 192.168.0.20 &> /dev/null && echo "OK" || echo "No"
OK
/ # ping -c1 -W1 192.168.0.1 &> /dev/null && echo "OK" || echo "No"
OK
/ # ping -c1 -W1 www.baidu.com &> /dev/null && echo "OK" || echo "No"
OK
```

**60节点**
```shell
/ # ping -c1 -W1 192.168.0.10 &> /dev/null && echo "OK" || echo "No"
OK
/ # ping -c1 -W1 192.168.0.1 &> /dev/null && echo "OK" || echo "No"
OK
/ # ping -c1 -W1 www.baidu.com &> /dev/null && echo "OK" || echo "No"
OK
```

## 方案二：创建子网卡实现多个虚拟网络同时访问公网

1.两个节点分别创建子接口
```
# apt-get -y install vlan
# vconfig add eth1 100
```

2. 两个节点分别配置子接口
```
# modprobe 8021q
# echo "8021q" >> /etc/modules
# vim /etc/network/interfaces
auto eth1.100
iface eth1.100 inet dhcp
    vlan-raw-device eth1  
```
3. 启动子接口，CTRL+C结束获取地址
```
# ifup eth1.100
```

4. 检查两个节点子接口状态，确保不是Down
```
root@node-50:/etc/udev/rules.d# ifconfig  eth1.100
eth1.100  Link encap:Ethernet  HWaddr 00:0c:29:96:82:40  
          inet6 addr: fe80::20c:29ff:fe96:8240/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:40 errors:0 dropped:0 overruns:0 frame:0
          TX packets:28 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:5750 (5.7 KB)  TX bytes:5500 (5.5 KB)
```

5. 两个节点分别创建网络
```
# docker network create -d macvlan --subnet=192.168.50.0/24 --gateway=192.168.50.1 -o parent=eth1.100  pub_net_50
```

6. 2节点创建各自容器
**50**
```
# docker run -it --name p_50_10 --net=pub_net_50 --ip=192.168.50.10 busybox sh
```

**60**
```
# docker run -it --name p_50_20 --net=pub_net_50 --ip=192.168.50.20 busybox sh
```

7. 测试
**50**
```
/ # ping -c1 -W1 192.168.50.20 &> /dev/null && echo "OK" || echo "No"
OK

/ # ping -c1 -W1 192.168.0.1 &> /dev/null && echo "OK" || echo "No"
No

```

**60**
```
# / # ping -c1 -W1 192.168.50.10 &> /dev/null && echo "OK" || echo "No"
OK

/ # ping -c1 -W1 192.168.0.1 &> /dev/null && echo "OK" || echo "No"
No
```

**失败！**
可以从测试结果看到，第一个需求跨主机容器通信OK，但是访问公网失败，唉！


