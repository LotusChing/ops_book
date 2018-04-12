# ZeroMQ
ZeroMQ是消息队列，简单理解和rmq相似


## Salt如何使用的ZeroMQ？
### 1. 发布/订阅
Pub/Sub 发布和订阅，默认监听4505端口。可以通过修改/etc/salt/master配置文件的publish_port参数设置。minion会主动连接master的tcp:4505端口(ZeroMQPubServerChannel进程)，而且是长连接，一直保持ESTABLISHED状态，时刻准备着接收master的消息



### 2. 请求/响应
2. Req/Rep 请求和响应，默认监听4506端口，可以通过修改/etc/salt/master配置文件的ret_port参数设置。它是salt客户端与服务端通信的端口。比如说Minion执行某个命令后的返回值就是发送给Master的4506这个REP端口


## Tips
### 显示salt-master进程名
```
apt-get install python-setproctitle -y
/etc/init.d/salt-master restart
```

查看salt-master进程名
```
root@server:/srv/salt# ps aux|grep salt
root      5941  1.5  0.1 317940 31864 ?        S    11:24   0:00 /usr/bin/python /usr/bin/salt-master -d ProcessManager
root      5942  0.0  0.1 316724 29616 ?        S    11:24   0:00 /usr/bin/python /usr/bin/salt-master -d MultiprocessingLoggingQueue
root      5943  0.0  0.1 325572 31468 ?        Sl   11:24   0:00 /usr/bin/python /usr/bin/salt-master -d ZeroMQPubServerChannel
root      5946  0.0  0.1 317376 30704 ?        S    11:24   0:00 /usr/bin/python /usr/bin/salt-master -d EventPublisher
root      5949 14.0  0.2 326624 38564 ?        S    11:24   0:00 /usr/bin/python /usr/bin/salt-master -d Maintenance
root      5950  0.5  0.1 318196 31116 ?        S    11:24   0:00 /usr/bin/python /usr/bin/salt-master -d ReqServer_ProcessManager
root      5951  0.0  0.2 621064 34116 ?        Sl   11:24   0:00 /usr/bin/python /usr/bin/salt-master -d MWorkerQueue
root      5958 49.0  0.2 336884 41832 ?        Sl   11:24   0:00 /usr/bin/python /usr/bin/salt-master -d MWorker-0
root      5959 49.5  0.2 336884 41416 ?        Sl   11:24   0:00 /usr/bin/python /usr/bin/salt-master -d MWorker-1
root      5960 49.0  0.2 336884 41412 ?        Sl   11:24   0:00 /usr/bin/python /usr/bin/salt-master -d MWorker-2
root      5961 48.5  0.2 336888 41412 ?        Sl   11:24   0:00 /usr/bin/python /usr/bin/salt-master -d MWorker-3
root      5962 49.0  0.2 337144 41416 ?        Sl   11:24   0:00 /usr/bin/python /usr/bin/salt-master -d MWorker-4
root      6529  0.0  0.0  10464   936 pts/20   S+   11:24   0:00 grep --color=auto salt
root     31836  0.0  0.2 481880 41048 ?        Sl   09:17   0:04 /usr/bin/python /usr/bin/salt-minion -d
root     31837  0.0  0.1 376992 24248 ?        S    09:17   0:00 /usr/bin/python /usr/bin/salt-minion -d

```

