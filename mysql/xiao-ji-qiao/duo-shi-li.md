# 多实例

其实哈，自从接触Docker之后就对MySQL的多实例一点兴趣都提不起来，因为我见过和学过的实现多实例的方式和Docker对比起来真的搓好多，不过考虑到现在很多地方还是CentOS6.x，万一有用到的时候呢，是吧~

## mysqld_multi

1. 准备数据目录，并初始化

    ```
    # cd /data/ && mkdir dbfile1 dbfile2
    # mysqld --initialize --user=mysql --datadir=/data/dbfile1
    # mysqld --initialize --user=mysql --datadir=/data/dbfile2
    ```
    
2. 修改配置文件
因为是虚拟机，内存不太够用，所以关闭了`performance_schema`，调低了`innodb_buffer_pool_size`
    ```
    # cat /etc/my.cnf
    [mysqld_multi]
    mysqld     = /usr/local/mysql/bin/mysqld_safe
    mysqladmin = /usr/local/mysql/bin/mysqladmin
    log        = /usr/local/mysql/multi.log
    
    [mysqld1]
    port                      = 3306
    bind_address              = 0.0.0.0
    log_error                 = /data/dbfile1/error.log
    datadir                   = /data/dbfile1
    socket                    = /data/dbfile1/mysql.sock    
    innodb_buffer_pool_size   = 32M
    performance_schema        = off
    character_set_server      = utf8mb4
    # skip-name-resolve
     
    [mysqld2]
    port                      = 3307
    bind_address              = 0.0.0.0
    log_error                 = /data/dbfile2/error.log
    datadir                   = /data/dbfile2
    socket                    = /data/dbfile2/mysql.sock
    innodb_buffer_pool_size   = 32M
    performance_schema        = off
    character_set_server      = utf8mb4
    # skip-name-resolve
    ```
3. 停止原mysqld服务(端口冲突的情况下)
    ```
    # /etc/init.d/mysqld stop
    ```

4. 使用`mysqld_multi`启动多实例
    启动单个实例，1为`my.cnf`中`mysqld`标签后的数字
    ```
    # mysqld_multi 1
    ```
    启动全部实例
    ```
    # mysqld_multi start
    ```
    
5. 查看多实例启动情况
    ```
    # mysqld_multi report
    Reporting MySQL servers
    MySQL server from group: mysqld1 is running
    MySQL server from group: mysqld2 is running
    ```
    
6. 连接各个mysql，因为默认没有`127.0.0.1`记录，而我们后期要禁用掉解析，所以手动创建一个
    ```
    # mysql -uroot -p -s /data/dbfile1/mysql.sock
    mysql> grant all on *.* to root@'127.0.0.1' identified by '复杂一点' with GRANT OPTION;
    mysql> flush privileges;
    ```
    另外一个
    ```
    # mysql -uroot -p -s /data/dbfile2/mysql.sock
    mysql> grant all on *.* to root@'127.0.0.1' identified by '复杂一点' with GRANT OPTION;
    mysql> flush privileges;
    ```

7. 停止多实例
    ```
    # mysqld_multi stop 1 (停止单个)
    # mysqld_multi stop (停止全部)
    ```
    
