# Salt 安装

## Ubuntu 14

1. 导入仓库公钥用以校验完整性
```shell
wget -O - https://repo.saltstack.com/apt/ubuntu/14.04/amd64/latest/SALTSTACK-GPG-KEY.pub | sudo apt-key add -
```

2. 添加salt源进source.list
```
deb http://repo.saltstack.com/apt/ubuntu/14.04/amd64/latest trusty main
```

3. 更新仓库
```
apt-get update
```

4. 安装salt
```
apt-get install salt-api
apt-get install salt-cloud
apt-get install salt-master
apt-get install salt-minion
apt-get install salt-ssh
apt-get install salt-syndic
```


## CentOS
1. 安装仓库
 ```
 # yum install https://repo.saltstack.com/yum/redhat/salt-repo- latest-2.el6.noarch.rpm
 ```

2. 清理缓存
 ```
 yum clean all
 ```
 
3. 安装salt和各组件
 ```
 sudo yum install salt-master
 sudo yum install salt-minion
 sudo yum install salt-ssh
 sudo yum install salt-syndic
 sudo yum install salt-cloud
 sudo yum install salt-api
 ```