# stub_status

配置文件

```bash
location /status {
    stub_status;
}
```
访问效果
```
Active connections: 4 
server accepts handled requests
6 6 12 
Reading: 0 Writing: 1 Waiting: 3 
```

第一行：
 * active connecttions意为当前活跃连接，这个值是reading+writing+waiting

第二行：
 * accepts: 已经接收到的连接数(1)
 * handled: 已经处理了的连接数(2)，如果handled和accepts不同，则说明当前服务器丢弃了一些连接
 * requests: 已经处理的请求数(3)，这个值通常会大于上面两个值，因为在默认http1.1版本，一个连接可以处理多个请求

第三行：
 * reading: 处于正在读取请求头的连接
 * writing: 处于正在写会响应头的连接
 * waiting: 开启keepalived时意为既没有读也没有写的连接
 
各项解释
活跃连接，标识当前连接数
握手次数、连接次数、总的请求数，正常情况下握手和连接数据相等，表示没有丢失
当前读的连接，正在写的连接，既没有读、也没有写处于空闲的连接(开启keepalive)