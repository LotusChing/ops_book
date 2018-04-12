# OS

## 后端服务器accept队列满

后端服务器`accept`队列满，导致后端服务器不回复`syn_ack`报文，客户端超时

解决方法：默认的`net.core.somaxconn`的值为`128`，执行`sysctl -w net.core.somaxconn=1024`更改它的值，并重启后端服务器上的应用。

## 文件句柄
`open_file_limits`：最大的打开文件句柄数


## 


