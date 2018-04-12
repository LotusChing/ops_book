# MySQL 免密

## 方法一：my.cnf

为了偷懒，可以在`my.cnf`中写上用户名密码
[mysql]
user=root
password=123123

虽然方便了，但是也危险了，如果服务器不是你一个人在用的话，`mysql root`的密码任何人都能看到了
```
# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 16
Server version: 5.7.18 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```


## 方法二：`--login-path`
通过`mysql_config_editor`来配置一个便捷登录标签
```
# mysql_config_editor set -G vml -u root -p
Enter password:
```

用户名密码信息是一二进制形式存储的，所以别人无法查看到
```
# mysql_config_editor print --all
[vml]
user = root
password = *****
```

尝试登录下
```
# mysql --login-path=vm1
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 18
Server version: 5.7.18 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```


除了mysql客户的，`mysqldump`、`mysqladmin`等等都支持`--login-path`使用标签


