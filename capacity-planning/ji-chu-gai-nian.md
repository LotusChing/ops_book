# 基础概念

## 并发数(Concurrency)和吞吐量(Throughput)
这个两个词放到不同的地方意义是不同的

在**网络设备**层面上来说，并发数指的是**网络设备所能支持的最大并发连接数**，吞吐量指的是在**不丢包的情况下经过网络设备的数据包的总长度**

在**Web**层面上来说，并发数指的是**系统能够同时处理的请求数**，吞吐量指的是**单位时间内处理请求的数量**

## QPS、TPS、RPS、RT

* QPS(`Queries per second`)：服务器每秒能够处理的查询请求数量(dns、db query)
* TPS(`Transaction per second`)：服务器每秒能够处理的事务数量(db)
* RPS(`Request per second`)：和QPS一回事
* RT(`Response time`)：从客户端发出一个请求开始，到服务器接收处理并将结果响应给客户端，总体花费的时间

## 



