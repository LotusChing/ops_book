# 返回程序

说白了就是定义执行结果返回到哪？我们不可能让执行结果返回到终端吧，那多搓啊，最起码返回到文件中，不过最好还是返回到mysql上，以便于以后查询

master->minion->mysql

1. master发出指令
2. minion接收并执行指令
3. minion写入执行结果到mysql

## minion 安装python mysql库
```
salt -G 'os:Ubuntu' cmd.run 'apt-get -y install python-mysqldb'
salt -G 'os:CentOS' cmd.run 'yum -y install MySQL-python'
```

## 创建salt库表并授权访问
直接复制官网的
```sql
CREATE DATABASE  `salt`
  DEFAULT CHARACTER SET utf8
  DEFAULT COLLATE utf8_general_ci;

USE `salt`;

CREATE TABLE `jids` (
  `jid` varchar(255) NOT NULL,
  `load` mediumtext NOT NULL,
  UNIQUE KEY `jid` (`jid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
CREATE INDEX jid ON jids(jid) USING BTREE;


CREATE TABLE `salt_returns` (
  `fun` varchar(50) NOT NULL,
  `jid` varchar(255) NOT NULL,
  `return` mediumtext NOT NULL,
  `id` varchar(255) NOT NULL,
  `success` varchar(10) NOT NULL,
  `full_ret` mediumtext NOT NULL,
  `alter_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  KEY `id` (`id`),
  KEY `jid` (`jid`),
  KEY `fun` (`fun`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE `salt_events` (
`id` BIGINT NOT NULL AUTO_INCREMENT,
`tag` varchar(255) NOT NULL,
`data` mediumtext NOT NULL,
`alter_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
`master_id` varchar(255) NOT NULL,
PRIMARY KEY (`id`),
KEY `tag` (`tag`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

授权访问
```sql
mysql> grant all on salt.* to 'salt'@'%' identified by 'salt@LotusChing';
```
## 配置minion
修改/etc/salt/minion，加入以下db配置，然后重启
```
mysql.host: 'salt'
mysql.user: 'salt'
mysql.pass: 'salt'
mysql.db: 'salt'
mysql.port: 3306
```


## 执行测试
使用`--return mysql`指定minion执行结果返回到mysql中
```
root@server:~# salt '*' cmd.run 'w' --return mysql
server:
     11:29:36 up 52 days,  1:12,  2 users,  load average: 0.26, 0.36, 0.34
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    root     pts/20   192.168.1.1      11:22   16.00s  0.73s  0.01s mysql -usalt -px xxxxxxxxxxxxx -h45.121.65.171 --port 3006 salt
    root     pts/4    192.168.1.1      09:04    0.00s  1.04s  0.31s /usr/bin/python /usr/bin/salt * cmd.run w --return mysql
iZ947mgy3c5Z:
     11:29:36 up 1 day,  2:20,  1 user,  load average: 0.09, 0.07, 0.05
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    root     pts/7    45.121.65.171    09:03    2:55   0.17s  0.17s -bash
```

查看mysql是否有数据
```
mysql> select * from salt_returns\G
*************************** 1. row ***************************
       fun: cmd.run
       jid: 20170929112936354283
    return: " 11:29:36 up 52 days,  1:12,  2 users,  load average: 0.26, 0.36, 0.34\nUSER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT\nroot     pts/20   192.168.1.1      11:22   16.00s  0.73s  0.01s mysql -usalt -px xxxxxxxxxxxxx -h45.121.65.171 --port 3006 salt\nroot     pts/4    192.168.1.1      09:04    0.00s  1.04s  0.31s /usr/bin/python /usr/bin/salt * cmd.run w --return mysql"
        id: server
   success: 1
  full_ret: {"fun_args": ["w"], "jid": "20170929112936354283", "return": " 11:29:36 up 52 days,  1:12,  2 users,  load average: 0.26, 0.36, 0.34\nUSER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT\nroot     pts/20   192.168.1.1      11:22   16.00s  0.73s  0.01s mysql -usalt -px xxxxxxxxxxxxx -h45.121.65.171 --port 3006 salt\nroot     pts/4    192.168.1.1      09:04    0.00s  1.04s  0.31s /usr/bin/python /usr/bin/salt * cmd.run w --return mysql", "retcode": 0, "success": true, "fun": "cmd.run", "id": "server"}
alter_time: 2017-09-29 11:29:36
*************************** 2. row ***************************
       fun: cmd.run
       jid: 20170929112936354283
    return: " 11:29:36 up 1 day,  2:20,  1 user,  load average: 0.09, 0.07, 0.05\nUSER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT\nroot     pts/7    45.121.65.171    09:03    2:55   0.17s  0.17s -bash"
        id: iZ947mgy3c5Z
   success: 1
  full_ret: {"fun_args": ["w"], "jid": "20170929112936354283", "return": " 11:29:36 up 1 day,  2:20,  1 user,  load average: 0.09, 0.07, 0.05\nUSER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT\nroot     pts/7    45.121.65.171    09:03    2:55   0.17s  0.17s -bash", "retcode": 0, "success": true, "fun": "cmd.run", "id": "iZ947mgy3c5Z"}
alter_time: 2017-09-29 11:29:37
2 rows in set (0.01 sec)
```

OK！大功告成！！！