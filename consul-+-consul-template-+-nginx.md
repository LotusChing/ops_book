# 使用Consul 实现服务动态伸缩


## 环境
* OS: Ubuntu 14.04.1 LTS, Trusty Tahr
* Kernel: 3.13.0-32-generic
* Docker: Docker version 17.05.0-ce, build 89658be

## 拓扑图
![![](/assets/consul_nginx_scaling.png)](/assets/consul_nginx_scaling.png)

拓扑图中有4个节点，各节点功能简单说下


* Consul K/V: 192.168.2.20，作为共享K/V存储，存放着overlay网络信息
* Consul Server：192.168.2.40，作为consul server服务注册中心，属于overlay网络中
* Server：192.168.2.50，作为应用服务器，测试时会在这个节点添加web容器测试是否回被正常调度，属于overlay网络
* Nginx：192.168.2.60，作为lb和应用服务器，60上会运行一个nginx容器，容器中会运行一个consul-template进程(daemon)，该进程会监控后端real server可用服务节点的状态变化，并当通过模版更新完nginx配置后，执行reload，初次之外，60也作为应用服务器，测试时会在这个节点添加web容器测试是否回被正常调度，60同样属于overlay网络


> 补充解释下这里为什么用到了两个consul server

> 首先，overlay网络通信要基于一个共享K/V存储，K/V存储中回存放着共享的网络信息，每个Overlay网络中的Docker节点都需要配置连接这个K/V存储

> 其次，为什么不使用40上的consul server即作为K/V存储又作为服务注册中心呢，最开始我确实是这么做的，但是，后来发现在获取服务节点client信息生成nginx配置文件时，upstream始终为空，后来仔细看了下，才知道consul server会定期的对client进行检测，端口是8301，如果检测失败例如超时，server会将该节点标识为critical不健康节点，所以nginx-template在生成nginx配置文件时获取不到健康的client信息，所以upstream始终为空。

> 最终，临时的解决方案就是单独出一个consul server仅作为共享的K/V存储，而要完成服务注册和检查的consul server加入overlay网络中，这样就可以正常的进行client注册和client健康检查，consul-template也可以正常的获取到健康的节点生成正常nginx配置文件


## 部署

### 1. Consul K/V共享存储
**1.1 下载consul包**
>下载页面：https://www.consul.io/downloads.html

```shell
wget --no-check-certificate https://releases.hashicorp.com/consul/1.0.0/consul_1.0.0_linux_amd64.zip?_ga=2.98354330.488929788.1509946081-1696779831.1509438596
```

**1.2 解压**
```
unzip consul_1.0.0_linux_amd64.zip  -d /bin/
```

**1.3 启动consul以server形式**
```
consul agent -server -ui -data-dir /tmp/consul -node=server -client 0.0.0.0 -bind=192.168.2.20 -advertise=192.168.2.20 -client 0.0.0.0  -bootstrap-expect=1
```
 * `-server`：以server角色启动
 * `-ui`：启动web界面
 * `-data-dir`：指定数据目录
 * `-node`：指定节点名称，默认时主机名
 * `-client`：指定绑定地址，用于客户端访问，例如RPC、DNS、HTTP、HTTPS
 * `-bind`：指定绑定的地址，用于集群节点通信，多网卡时需要指定，单网卡时默认选择第一个私有IP进行绑定监听
 * `-bootstrap-expect`：集群中server节点的数量

**1.4 检查输出并查看ui界面**
> ui地址：192.168.2.20:8500，以下时启动时正常的输出

```
BootstrapExpect is set to 1; this is the same as Bootstrap mode.
bootstrap = true: do not enable unless necessary
==> Starting Consul agent...
==> Consul agent running!
           Version: 'v1.0.0'
           Node ID: '3f487bab-6885-d314-d0f1-1115561804c8'
         Node name: 'server'
        Datacenter: 'dc1' (Segment: '<all>')
            Server: true (Bootstrap: true)
       Client Addr: [0.0.0.0] (HTTP: 8500, HTTPS: -1, DNS: 8600)
      Cluster Addr: 192.168.2.20 (LAN: 8301, WAN: 8302)
           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false

==> Log data will now stream in as it occurs:

    2017/11/02 16:21:30 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:3f487bab-6885-d314-d0f1-1115561804c8 Address:192.168.2.20:8300}]
    2017/11/02 16:21:30 [INFO] serf: EventMemberJoin: server.dc1 192.168.2.20
    2017/11/02 16:21:30 [INFO] raft: Node at 192.168.2.20:8300 [Follower] entering Follower state (Leader: "")
    2017/11/02 16:21:30 [WARN] serf: Failed to re-join any previously known node
    2017/11/02 16:21:30 [INFO] serf: EventMemberJoin: server 192.168.2.20
    2017/11/02 16:21:30 [INFO] agent: Started DNS server 0.0.0.0:8600 (udp)
    2017/11/02 16:21:30 [WARN] serf: Failed to re-join any previously known node
    2017/11/02 16:21:30 [INFO] consul: Adding LAN server server (Addr: tcp/192.168.2.20:8300) (DC: dc1)
    2017/11/02 16:21:30 [INFO] consul: Handled member-join event for server "server.dc1" in area "wan"
    2017/11/02 16:21:30 [INFO] agent: Started DNS server 0.0.0.0:8600 (tcp)
    2017/11/02 16:21:30 [INFO] agent: Started HTTP server on [::]:8500 (tcp)
    2017/11/02 16:21:36 [WARN] raft: Heartbeat timeout from "" reached, starting election
    2017/11/02 16:21:36 [INFO] raft: Node at 192.168.2.20:8300 [Candidate] entering Candidate state in term 4
    2017/11/02 16:21:36 [INFO] raft: Election won. Tally: 1
    2017/11/02 16:21:36 [INFO] raft: Node at 192.168.2.20:8300 [Leader] entering Leader state
    2017/11/02 16:21:36 [INFO] consul: cluster leadership acquired
    2017/11/02 16:21:36 [INFO] consul: New leader elected: server
    2017/11/02 16:21:36 [INFO] agent: Synced node info
```

默认时运行在前台，可以使用`nohup `让其运行在后台

### 2. Docker配置使用共享K/V存储
> 官方文档：https://docs.docker.com/engine/security/https/#create-a-ca-server-and-client-keys-with-openssl

**2.1 Docker Daemon为什么要配置TLS**
> 首先先解释下，为什么要配置TLS，默认情况docker是通过Unix套接字方式运行，Docker也支持http方式运行，启动Docker Daemon时，加入-H 0.0.0.0:2375，Docker Daemon就可以接收远端的Docker Client发送的指令。注意，Docker是把2375端口作为非加密端口暴露出来，这种方式时极其不安全的。此时，没有任何加密和认证过程，只要知道Docker主机的IP，任何人都可以管理这台主机上的容器和镜像。

**2.2 使用OpenSSL创建CA、服务器端、客户端密钥**

生成CA公私钥
```
openssl genrsa -aes256 -out ca-key.pem 4096
```

创建证书请求(csr)
```
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
# Common Name填Docker主机名
Common Name (e.g. server FQDN or YOUR name) []:$HOST
```

创建服务器公私钥
```
openssl genrsa -out server-key.pem 4096
```

创建服务端证书请求，HOST填写主机名
```
openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr
```

由于TLS连接可以通过IP地址和DNS名称进行，因此创建证书时需要指定IP地址。例如，要允许使用192.168.2.0和127.0.0.1的连接：

```
echo subjectAltName = DNS:$HOST,IP:192.168.2.0,IP:127.0.0.1 >> extfile.cnf
```

设置为仅服务器验证
```
echo extendedKeyUsage = serverAuth >> extfile.cnf
```

签发证书
```
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
```

创建客户端公私钥和证书请求
```
openssl genrsa -out key.pem 4096
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
```

设置客户端验证
```
echo extendedKeyUsage = clientAuth >> extfile.cnf
```

签发证书
```
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile.cnf
```

删除证书请求
```
rm -v client.csr server.csr
```

设置密钥、证书权限
```
chmod -v 0400 ca-key.pem key.pem server-key.pem
chmod -v 0444 ca.pem server-cert.pem cert.pem
```


**2.3 启动docker服务**
这里启动两种，第一种是使用命令行(官方文档)`dockerd --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem -H=0.0.0.0:2376`

很明显这种方式不是很好，所以这里使用docker配置文件`/etc/docker/daemon.json`
```
cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://hz6lwzkb.mirror.aliyuncs.com"],
  "tlsverify": true,
  "hosts": ["tcp://0.0.0.0:2376", "unix:///var/run/docker.sock"], 
  "tlscacert": "/root/ca.pem",
  "tlscert": "/root/server-cert.pem",
  "tlskey": "/root/server-key.pem",
  "cluster-store": "consul://192.168.2.20:8500",
  "cluster-advertise": "192.168.2.40:2376"
}
```

启配置完成后重启服务，清空日志方便查看启动日志
```
service docker stop
>/var/log/upstart/docker.log
service docker start
```

**2.4 检查配置是否成功**

浏览器打开 192.168.2.20:8500 > KEY/VALUE > docker > nodes/ ，不出意外就能看到docker节点信息了
![](/assets/deploy_40_in_20.png)

**2.5 其余节点(50, 60)配置tls**
步骤是一样的，最后到192.168.2.20:8500检查下是否能看到3台节点，如果能看到就表示正常


### 3. 创建overlay网络
3.1 创建网络
3.2 测试连通性

### 4. 创建consul 服务注册中心

### 5. Dockerfile编写
4.1 app容器Dockerfile
4.2 nginx容器Dockerfile编写

### 6. 测试动态伸缩
6.1 添加容器
6.2 删除容器

