# Network
评估网络的五个指标
* 带宽(`Bandwidth`)：最大吞吐量
* 吞吐量(`Throughput `)：实际吞吐量
* 延迟(`Latency`)：发送方和接收方传输和解码的时间
* 抖动(`Jitter`)：传输中不定期的物理链路不稳定
* 丢包率(`Error rate`)：传输中错误/丢失的数据比率

## Iperf
```
# rpm -ivh https://iperf.fr/download/fedora/iperf-2.0.1-1.2.el4.rf.x86_64.rpm
```


### 带宽
服务器端运行
```
Server# iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 192.168.2.30 port 5001 connected with 192.168.2.20 port 38317
[  4]  0.0-10.0 sec    664 MBytes    557 Mbits/sec
```


客户端运行
```
# iperf -c 192.168.2.30
------------------------------------------------------------
Client connecting to 192.168.2.30, TCP port 5001
TCP window size: 23.2 KByte (default)
------------------------------------------------------------
[  3] local 192.168.2.20 port 38317 connected with 192.168.2.30 port 5001
[  3]  0.0-10.0 sec    664 MBytes    557 Mbits/sec
```


Server端修改有关TCP窗口内核参数
```
Server# echo 'net.core.wmem_max=4194304' >> /etc/sysctl.conf 
Server# echo 'net.core.rmem_max=12582912' >> /etc/sysctl.conf 
Server# echo 'net.ipv4.tcp_rmem = 4096 87380 4194304' >> /etc/sysctl.conf 
Server# echo 'net.ipv4.tcp_wmem = 4096 87380 4194304' >> /etc/sysctl.conf 
Server# sysctl -p
```

服务器端
```
[root@Da ~]# iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  5] local 192.168.2.30 port 5001 connected with 192.168.2.20 port 38320
[  5]  0.0-10.0 sec    807 MBytes    677 Mbits/sec
```


客户端
```
# iperf -c 192.168.2.30 
------------------------------------------------------------
Client connecting to 192.168.2.30, TCP port 5001
TCP window size: 23.2 KByte (default)
------------------------------------------------------------
[  3] local 192.168.2.20 port 38320 connected with 192.168.2.30 port 5001
[  3]  0.0-10.0 sec    807 MBytes    677 Mbits/sec
```


可以看到TCP滑动窗口对性能的影响非常大，从每秒69M提升到每秒84M，啧啧~
