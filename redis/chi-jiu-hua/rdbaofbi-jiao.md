# RDB、AOF比较和推荐配置

## RDB、AOF启动载入数据顺序
草图
![](/assets/aof_rbd_start_load_data_flow.jpg)


流程比较简单，补充说明下
通过载入RDB文件启动，日志输出
```
30721:M 04 Jan 11:48:52.938 * DB loaded from disk: 0.000 seconds
30721:M 04 Jan 11:48:52.938 * The server is now ready to accept connections on port 6379
```

通过载入AOF文件启动，日志输出
```
30763:M 04 Jan 11:49:38.196 * DB loaded from append only file: 0.014 seconds
30763:M 04 Jan 11:49:38.196 * The server is now ready to accept connections on port 6379
```

如果没有rbd、aof数据文件，则日志中不会有任何load数据说明，只有服务端在指定的端口准备接收连接
```
31755:M 04 Jan 13:21:53.196 * The server is now ready to accept connections on port 6379
```

## RDB、AOF比较

|类型|RDB|AOF|
|:--:|:-:|:-:|
|启动优先级|低|高|
|文件体积|小|大|
|恢复速度|块|慢|
|数据安全性|丢数据|取决于策略|
|系统影响|重|轻(不考虑rewrite，仅追加)|


## RDB推荐配置
* "关"，或者控制save频率
* 远程存储，集中管理


## AOF推荐配置
* 开，或者不开，取决于应用场景，两个判断条件，1. 数据是否重要，2. 回源代价大不大
* 远程存储，集中管理
* everysec

## 最佳实践
* 多实例、小分片
* 监控硬盘、CPU、内存、负载、网络