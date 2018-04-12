# Audit in MySQL 5.7
由于`mysql 5.7`社区版不支持`audit`功能，所以如果有类似需求的话，可以尝试下`MariaDB`的审计插件

## 环境

**!!!注意!!!**，不要在centos 6.x下使用，因为MariaDB的audit插件依赖glibc 2.14版本以上，而默认centos 6.x的版本是2.12，glibc尽量不要乱搞，会把系统搞爆炸的

所以俺的OS环境是`Ubuntu 14.04`，`Ubuntu`的`glibc`默认版本是`2.19`


## 操作
1. 下载mariaDB的包，然后把`server_audit.so`弄出来，我是直接起了个MariaDB的容器，然后docker cp出来的
```
# docker run --name mariadb -e MYSQL_ROOT_PASSWORD=123123 -d mariadb
# docker cp mariadb:/usr/lib/mysql/plugin/server_audit.so .
# sz server_audit.so
```

2. 上传`server_audit.so`到mysql插件目录下
```
mysql> show global variables like 'plugin_dir';
+---------------+------------------------------+
| Variable_name | Value                        |
+---------------+------------------------------+
| plugin_dir    | /usr/local/mysql/lib/plugin/ |
+---------------+------------------------------+
# cd /usr/local/mysql/lib/plugin/
# rz -be
# chmod +x server_audit.so
```

3. 安装审计插件
执行`show plugins`就可以看到审计插件信息了
```
mysql> INSTALL PLUGIN server_audit SONAME 'server_audit.so';
Query OK, 0 rows affected (0.01 sec)
mysql> show plugins;
```

4. 永久生效
写到配置文件后，每次启动mysql会自动载入插件，配置文件中还有几条是测试的参数，具体详细的参数在[官方文档](https://mariadb.com/kb/en/mariadb/about-the-mariadb-audit-plugin/)里

    ```
    # cat /etc/mysql/my.cnf
    [mysql]
    user=root
    password=123123
    socket=/data/dbfile/mysql.sock
    
    [mysqld]
    plugin-load=server_audit=server_audit.so
    server_audit_logging=1
    server_audit_events=connect,query
    datadir=/data/dbfile
    log_error =/data/dbfile/error.log
    socket=/data/dbfile/mysql.sock
    ```
5. 测试
```
# /etc/init./mysql57 restart
# cat /data/dbfile/server_audit.log
20170629 16:53:40,Lotus,root,localhost,3,0,CONNECT,,,0
20170629 16:53:40,Lotus,root,localhost,3,2,QUERY,,'select @@version_comment limit 1',0
20170629 16:54:03,Lotus,root,localhost,3,3,QUERY,,'show global variables like \'%server_audit_logging%\'',0
20170629 16:54:09,Lotus,root,localhost,3,4,QUERY,,'show global variables like \'%server_audit_%\'',0
20170629 16:54:17,Lotus,root,localhost,3,0,DISCONNECT,,,0
20170629 17:10:10,Lotus,root,localhost,4,0,CONNECT,,,0
20170629 17:10:10,Lotus,root,localhost,4,6,QUERY,,'select @@version_comment limit 1',0
20170629 17:10:15,Lotus,root,localhost,4,7,QUERY,,'show global variables like \'%plugin%\'',0
20170629 17:10:22,Lotus,root,localhost,4,8,QUERY,,'show global variables like \'plugin_dir\'',0
20170629 17:16:44,Lotus,root,localhost,4,0,DISCONNECT,,,0
```