# Docker方式

最快速的方式毫无疑问就是根据compose文件直接启动，不过启动之前有几个地方需要提前准备下

## 准备工作一：创建并授权数据库访问帐号
```sql
mysql> create database zabbix charset utf8;
mysql> grant all on zabbix.* to 'zabbix'@'%' identified by 'zabbix@lotus';
```

## 准备工作二：检查网络连通性
* iptable
* 安全组
* 白名单
* ...

## 准备工作三：安装docker-compose
参考笔记：https://lotusching.gitbooks.io/ops/content/jin-jie/compose.html

## 准备工作思：检查docker版本
根据当前dockerb版本修改version字段，同时较老版本会不支持bind方式挂载，需要对应修改，旧版本volumes配置
```yaml
services:
  elk_slave:
    volumes:
      -  /src:/dst
```


## 启动zabbix服务

```
root@VM-128-6-ubuntu:~# cat /prodata/scripts/zabbix/docker-compose.yml 
```
```yaml
version: '3.2'
services:
  zabbix-server:
    image: zabbix/zabbix-server-mysql:ubuntu-3.4-latest
    container_name: zabbix-server-1
    ports:
      - "10051:10051"
    environment:
      - DB_SERVER_HOST=1.1.1.1
      - DB_SERVER_PORT=3000
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=zabbix
      - TZ=Asia/Shanghai
    volumes:
      - type: bind
        source: /opt/zabbix-server/etc/zabbix_server.conf
        target: /etc/zabbix/zabbix_server.conf
      - type: bind
        source: /opt/zabbix-server/alertscripts
        target: /usr/lib/zabbix/alertscripts
  zabbix-web:
    container_name: zabbix-web-1
    image: zabbix/zabbix-web-nginx-mysql:ubuntu-3.4-latest
    ports:
      - "10080:80"
    environment:
      - DB_SERVER_HOST=1.1.1.1
      - DB_SERVER_PORT=3000
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=zabbix
      - ZBX_SERVER_HOST=zabbix-server
      - PHP_TZ=Asia/Shanghai
      - TZ=Asia/Shanghai
    volumes:
      - type: bind
        source: /opt/zabbix-web/zabbix
        target: /usr/share/zabbix
    links:
      - zabbix-server
```
```docker
# docker-compose up -d
```

## 补充说明
* agent端不要使用容器来运行，因为容器是隔离的，所以导致很多指标的不准确，无法反映宿主机的真实运行情况，例如当前用户数等

* 如果compose中service发送变更，单独重建service
```docker
# docker-compose up -d --no-deps --build zabbix-web
```

* 中文支持：参考笔记常见问题/中文乱码
