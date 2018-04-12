# 环境安装

## 方法一：通过包管理器安装

### Ubuntu
**安装OpenJDK7**
```shell
# apt-get -y install openjdk-7-jdk
```

**安装OpenJDK8**

安装openjdk8，需要添加库，但是这个库在某些网络环境下可能是访问不了的，我当时是办公环境访问不了，阿里云可以，所以在阿里云用nginx做了http正向代理
```nginx
    server {
        listen        88;
        location / {
            resolver 8.8.8.8;
            proxy_pass $scheme://$http_host$request_uri;
        }
    }
```

如果长时间卡在add-apt-repository说明当前网络无法添加远程库
```shell
# apt-get -y install software-properties-common
# add-apt-repository ppa:openjdk-r/ppa

```
这条添加仓库的命令其实就是在sources.list中添加两行记录，配置使用完正向代理后，手动添加这两行记录，**注意YOUR_UBUNTU_VERSION_HERE是Ubuntu的版本，可以通过`# lsb_release -c -s`命令获取**
```shell
# export http_proxy=http://ip:port/
# cat /etc/apt/sources.list
...
deb http://ppa.launchpad.net/openjdk-r/ppa/ubuntu YOUR_UBUNTU_VERSION_HERE main 
deb-src http://ppa.launchpad.net/openjdk-r/ppa/ubuntu YOUR_UBUNTU_VERSION_HERE main 
...
# apt-get update
# apt-get install openjdk-8-jdk
```

如果需要修改java和javac的版本，执行下面命令
```shell
# update-alternatives --config java
# update-alternatives --config javac
```








**安装Oracle-java8**
大体和安装openjdk-8很像，都是添加库，然后安装对应的包，如果有需要修改默认的版本就行了
```shell
# add-apt-repository ppa:webupd8team/java -y
# apt-get update
# apt-get install oracle-java8-installer
```



## 方法二：下载解压安装
