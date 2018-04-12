# 负载均衡
## VIP
Swarm内部有一个DNS，service名称和容器名称都可以被解析，由于Swarm默认会做LVS，通过Service名称解析到就是VIP，然后根据RR策略进行负载均衡
```docker
root@node-40:~# docker service inspect rs_nginx --pretty|grep Endpoint
Endpoint Mode:	vip
```

这里起了测试的Service，安装下curl，然后测试访问并观察日志检查是否LB~
```docker
root@node-40:~# docker service create --name client --network overlay_net httpd ping 192.168.1.1
root@node-60:~# docker exec -it client.1.wri3tddia85vzbkrh4npbcvs2 bash
root@915c798f398b:/usr/local/apache2# apt-get update
root@915c798f398b:/usr/local/apache2# curl
root@915c798f398b:/usr/local/apache2# apt-get install curl
root@915c798f398b:/usr/local/apache2# curl -I rs_nginx
```

## DNS

创建时配置为DNS轮训
```docker
root@node-40:~# docker service create --replicas 3 --name rs_nginx_dnsrr --network overlay_net --endpoint-mode dnsrr nginx:1.12
e81zlf4qrt45dxhyzhvhvxeo5
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 
```

测试
```shell
root@915c798f398b:/usr/local/apache2# ping -c1 -W1 rs_nginx_dnsrr -q
PING rs_nginx_dnsrr (10.0.0.40): 56 data bytes
--- rs_nginx_dnsrr ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max/stddev = 1.552/1.552/1.552/0.000 ms
root@915c798f398b:/usr/local/apache2# ping -c1 -W1 rs_nginx_dnsrr -q
PING rs_nginx_dnsrr (10.0.0.42): 56 data bytes
--- rs_nginx_dnsrr ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max/stddev = 1.530/1.530/1.530/0.000 ms
root@915c798f398b:/usr/local/apache2# ping -c1 -W1 rs_nginx_dnsrr -q
PING rs_nginx_dnsrr (10.0.0.41): 56 data bytes
```


修改现有VIP轮询为DNS轮询，由于DNS轮训不支持对外暴露端口，所以首先要删除开放端口，然后修改调度方式为DNS轮询
```docker
root@node-40:~# docker service update --publish-rm 80 rs_nginx 
root@node-40:~# docker service update --endpoint-mode dnsrr rs_nginx 
```

测试
```shell
root@915c798f398b:/usr/local/apache2# ping -c1 -W1 rs_nginx -q
PING rs_nginx (10.0.0.38): 56 data bytes
--- rs_nginx ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max/stddev = 1.321/1.321/1.321/0.000 ms
root@915c798f398b:/usr/local/apache2# ping -c1 -W1 rs_nginx -q
PING rs_nginx (10.0.0.36): 56 data bytes
--- rs_nginx ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max/stddev = 2.212/2.212/2.212/0.000 ms
root@915c798f398b:/usr/local/apache2# ping -c1 -W1 rs_nginx -q
PING rs_nginx (10.0.0.37): 56 data bytes
--- rs_nginx ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.754/0.754/0.754/0.000 ms
```


解析记录
```
root@915c798f398b:/usr/local/apache2# dig rs_nginx
...
rs_nginx.		600	IN	A	10.0.0.36
rs_nginx.		600	IN	A	10.0.0.37
rs_nginx.		600	IN	A	10.0.0.38
rs_nginx.		600	IN	A	10.0.0.5
...
```
