# 配置、启动、关闭

Redis 启动方式有三种

## 启动 Redis
**1. 默认配置启动**
 ```
 [root@bj-vmware-test1 redis]# redis-server
 58421:C 22 Dec 09:03:33.479 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                 _._                                                  
            _.-``__ ''-._                                             
       _.-``    `.  `_.  ''-._           Redis 3.0.7 (00000000/0) 64 bit
   .-`` .-```.  ```\/    _.,_ ''-._                                   
  (    '      ,       .-`  | `,    )     Running in standalone mode
  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
  |    `-._   `._    /     _.-'    |     PID: 58421
   `-._    `-._  `-./  _.-'    _.-'                                   
  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
  |    `-._`-._        _.-'_.-'    |           http://redis.io        
   `-._    `-._`-.__.-'_.-'    _.-'                                   
  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
  |    `-._`-._        _.-'_.-'    |                                  
   `-._    `-._`-.__.-'_.-'    _.-'                                   
       `-._    `-.__.-'    _.-'                                       
           `-._        _.-'                                           
               `-.__.-'                                               
 
 58421:M 22 Dec 09:03:33.492 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
 58421:M 22 Dec 09:03:33.492 # Server started, Redis version 3.0.7
 58421:M 22 Dec 09:03:33.492 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
 58421:M 22 Dec 09:03:33.494 * The server is now ready to accept connections on port 6379
 ```
 
** 2. 启动时配置参数**
  ```
  [root@bj-vmware-test1 redis]# redis-server --port 6380
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.0.7 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6380
 |    `-._   `._    /     _.-'    |     PID: 58426
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               
 
 58426:M 22 Dec 09:04:44.284 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
 58426:M 22 Dec 09:04:44.284 # Server started, Redis version 3.0.7
 58426:M 22 Dec 09:04:44.284 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
 58426:M 22 Dec 09:04:44.285 * DB loaded from disk: 0.000 seconds
 58426:M 22 Dec 09:04:44.285 * The server is now ready to accept connections on port 6380
  ```
  
**3. 配置文件启动(推荐)**
 ```
 [root@bj-vmware-test1 redis]# redis-server /opt/redis/redis.conf 
                 _._                                                  
            _.-``__ ''-._                                             
       _.-``    `.  `_.  ''-._           Redis 3.0.7 (00000000/0) 64 bit
   .-`` .-```.  ```\/    _.,_ ''-._                                   
  (    '      ,       .-`  | `,    )     Running in standalone mode
  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6380
  |    `-._   `._    /     _.-'    |     PID: 58435
   `-._    `-._  `-./  _.-'    _.-'                                   
  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
  |    `-._`-._        _.-'_.-'    |           http://redis.io        
   `-._    `-._`-.__.-'_.-'    _.-'                                   
  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
  |    `-._`-._        _.-'_.-'    |                                  
   `-._    `-._`-.__.-'_.-'    _.-'                                   
       `-._    `-.__.-'    _.-'                                       
           `-._        _.-'                                           
               `-.__.-'                                               
 
 58435:M 22 Dec 09:06:38.672 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
 58435:M 22 Dec 09:06:38.672 # Server started, Redis version 3.0.7
 58435:M 22 Dec 09:06:38.672 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
 58435:M 22 Dec 09:06:38.672 * DB loaded from disk: 0.000 seconds
 58435:M 22 Dec 09:06:38.672 * The server is now ready to accept connections on port 6380
 ```
 
## 配置 Redis
`redis.conf`中很多配置，基础的几个配置
* port: Redis 监听端口，默认6379，**生产中建议修改端口**
* logfile: 日志文件，默认redis会输出到终端，如果以daemon运行，则输出到/dev/null
* dir: Redis工作目录(存放持久化文件、日志文件)，默认是当前目录，建议创建一个data目录
* daemonize: 是否后台运行，默认前台运行

修改后的配置
```shell
# egrep "^port|^logfile|^dir|^daemonize" redis.conf 
daemonize yes
port 6379
logfile "redis.log"
dir ./data
```

 

## 关闭 Redis
大多数老哥好像都是kill，但可以通过redis-cli执行关闭
```
# redis-cli shutdown
```

查看日志可以看到，redis接收到关闭的请求后，首先把数据进行了一次刷盘，确认save到磁盘后，在进行关闭的，所以通过`redis-cli shutdown`相对更安全一点
```
[root@bj-vmware-test1 redis]# tail data/redis.log
...
58493:M 22 Dec 09:19:10.129 # User requested shutdown...
58493:M 22 Dec 09:19:10.129 * Saving the final RDB snapshot before exiting.
58493:M 22 Dec 09:19:10.143 * DB saved on disk
58493:M 22 Dec 09:19:10.143 * Removing the pid file.
58493:M 22 Dec 09:19:10.143 # Redis is now ready to exit, bye bye...
```


 