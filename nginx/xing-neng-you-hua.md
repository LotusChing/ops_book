# 性能优化

## 通用配置
### 文件句柄
`limits.conf`可以限制全部用户或者特定用户的文件句柄数
```
cat /etc/security/limits.conf
...
root soft nofile 65535
root hard nofile 65535
*    soft nofile 65535
*    hard nofile 65535
```

如果想实现针对进程的打开文件句柄数，main段使用`worker_rlimit_nofile`来限制
```
# 最大Nginx打开文件句柄
worker_rlimit_nofile 35525;
```
1. **什么是文件句柄？**
2. **调整文件句柄的好处？**
3. **那些地方配置文件句柄限制？限制的范围有哪些？**


### CPU亲和
降低worker进程在不同CPU核上切换的频率以避免不必要的性能损耗
```
worker_process auto | 4;
worker_cpu_affinity auto | 0001 0010 0100 1000;
```

1. **worker_process的作用？**
2. **worker_process是不是越大越好？为什么？**
3. **CPU亲和原理和作用？**
4. **CPU亲和有几种配置？**



### worker_connections
```
events {
    use                 epoll;
    worker_connections  10240;
}
```
1. **worker_connections作用**
2. **worker_connections过大过小有什么问题？**

### sendfile & tcp_nopush & tcp_nodelay
```
    sendfile           on;
    tcp_nopush         on;
    #tcp_nodelay        on;
    keepalive_timeout  65;
```

1. **sendfile是什么？开启后有哪些好处？ 使用场景是哪？**
2. **tcp_nopush是什么？ 开启后有那些好处？ 适用场景是哪？**
3. **tcp_nodelay是什么？ 开启后有哪些好处？ 适用场景是哪？**

### gzip
```
     gzip               on;
     gzip_disable       "MSIE [1-6]\."
```
1. **gzip压缩是什么，开启后好处有哪些？ 不同文件的压缩情况如何？ **
2. **gzip压缩的问题有哪些？ 如何规避这些问题？**

