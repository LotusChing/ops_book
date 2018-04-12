# Nginx+Tomcat+MySQL


## 实验前疑问

VIP和DNSRR调度时是否会检查容器的健康状态

* Nginx LB：3个容器
* Tomcat App：3个容器
* MySQL:3个容器

GLB -> Nginx -> Tomcat -> MySQL

## Nginx
* 通过应用配置文件存储实现配置文件管理
* 反向代理Tomcat时通过VIP或DNSRR


## Tomcat
* 挂在volume时使用NFS远程卷挂载，保证容器不写入数据到容器内部，主要是webapps和logs两个目录
* 通过DNS连接读库、写库
