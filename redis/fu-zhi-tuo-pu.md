# 复制拓扑

## 一主一/多丛

master节点作为写节点
slave作为读节点

### 特点
* slave节点故障影响可控，手动/自动切换到正常的slave节点进行读取数据


### 故障处理
#### slave故障
虽然这这里解决了读数据的高可用，但是有几个点需要注意
1. 其余正常的slave节点能否承受住读请求的压力 
2. 如果客户端只是一个app，那修改下正常slave节点地址，然后重启app应用新slave节点地址，这样还算简单，如果是几十个或者更多，那就很麻烦了，所以，建议首先利用Consul的dns功能通过域名连接redis，例如r.rds.domain.com，然后通过Consul的健康检查机制实现动态删除故障节点，故障恢复后动态恢复到可用列表中

#### master故障处理
1. 找一个写性能较好的slave节点作为master，执行`slaveof no one`
2. 其余slave节点`slaveof master_ip master_port`



### 问题
* master节点故障不可控

### 总结
* **没有实现故障自动转移**

### 补充阅读
http://ningg.top/redis-lesson-8-redis-master-slave/
