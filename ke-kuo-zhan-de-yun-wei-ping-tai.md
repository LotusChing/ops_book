# 可扩展的运维平台

## 架构

域名 -> SLB -> Web/App服务 -> Cache/DB服务

## 目录结构
```python
root@blue-db-master:/var/project/blue# tree -L 2
.
├── app
│   ├── blue
│   ├── docker-compose.yml
│   └── nginx
├── cache
│   └── docker-compose.yml
└── data
    └── docker-compose.yml
```

## DB/Cache 关键文件
### DB
创建DB服务，然后导入初始数据，这里虽然对外暴露了端口，但是用的都是内网的地址，这里最好用内部DNS域名，方便以后变更
**data/docker-compose.yml**
```yaml
root@blue-db-master:/var/project/blue# cat data/docker-compose.yml 
version: "3.3"
services:
  db:
    image: mysql:5.6
    ports:
      - "3000:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=LotusChing
      - MYSQL_DATABASE=ops_v2
      - MYSQL_USER=lotus
      - MYSQL_PASSWORD=ching     
    volumes:
      - type: bind
        source: /prodata/data/db
        target: /var/lib/mysql
```

### Cache
**cache/docker-compose.yml**
```yaml
root@blue-db-master:/var/project/blue# cat cache/docker-compose.yml 
version: '3.3'
services:
  redis:
    image: redis
    volumes:
      - type: bind
        source: /prodata/data/redis
        target: /data
    ports:
      - "6379:6379"
```


## Web/APP 关键文件
### Web
**app/docker-compose.yml**
```yaml
root@blue-db-master:/var/project/blue# cat app/docker-compose.yml 
version: '3.3'
services:
  lb:
    build: ./nginx
    image: registry-vpc.cn-qingdao.aliyuncs.com/toybox/blue:lb_0.1
    ports:
      - "80:80"
    networks:
      - frontend
  web:
    build: ./blue
    image: registry-vpc.cn-qingdao.aliyuncs.com/toybox/blue:app_0.1
    environment:
      - DB_HOST=172.31.117.186
      - DB_PORT=3000
      - DB_USERNAME=lotus
      - DB_PASSWORD=ching
      - DB_NAME=ops_v2
      - REDIS_HOST=172.31.117.186
      - REDIS_PORT=6379
    ports:
      - "9090:9090"
    networks:
      - frontend

networks:
  frontend:
    driver: overlay
```

**app/nginx/Dockerfile**
```docker
root@blue-db-master:/var/project/blue# cat app/nginx/Dockerfile 
FROM centos:6
MAINTAINER LotusChing

RUN cd /etc/yum.repos.d && mkdir bak && mv *.repo bak
ADD nginx.repo /etc/yum.repos.d
RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
RUN yum -y install openssl-devel pcre-devel nginx

# locales
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
RUN echo "export LC_ALL=en_US.UTF-8" >> ~/.bashrc
RUN echo "export LANG=en_US.UTF-8" >> ~/.bashrc
RUN echo "export LANGUAGE=en_US.UTF-8" >> ~/.bashrc

# timezone
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
RUN echo "Asia/Chongqing" > /etc/timezone
ENV TZ Asia/Shanghai


ADD nginx.conf /etc/nginx/conf.d/default.conf
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
```

**app/nginx/nginx.conf**
```nginx
root@blue-db-master:/var/project/blue# cat app/nginx/nginx.conf 
upstream apps{
    server web:9090;
}
server {
    listen    80;
    server_name  localhost;
    root      html;
    index     index.html index.php;

    location / {
        proxy_pass http://apps;
    }
}
```

**app/nginx/nginx.repo**
```python
root@blue-db-master:/var/project/blue# cat app/nginx/nginx.repo 
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/6/$basearch/
gpgcheck=0
enabled=1
```

**app/blue/Dockerfile**
```docker
root@blue-db-master:/var/project/blue# cat app/blue/Dockerfile 
FROM python:3.5
ADD . /code
ADD pip.conf /root/.pip/pip.conf
WORKDIR /code
RUN pip install -r requirements.txt
CMD uwsgi --ini uwsgi.ini
```

## 创建Swarm集群
初始化集群，并且设置manager节点不处理Task任务
```shell
root@blue-db-master:~# docker swarm init --advertise-addr <内网地址>
root@blue-db-master:~# docker node update --availability drain blue-db-master 
```

2个worker节点
```shell
root@blue-db-master:~# docker swarm join --token SWMTKN-1-4gqdvsc4zqmkk8c3yq4tiwgu5rp2npidz9fi8eb9zwx94moew2-4c2s556h5lszw5a0nxef1bqk4 <内网地址:端口>
```

## Cache/Data部署
**由于DB扩容并不像Web能够简单的扩容，所以这以compose方式启动**

启动DB服务，然后导入数据
```shell
root@blue-db-master:/var/project/blue# cd /var/project/blue/data/
root@blue-db-master:/var/project/blue/data# docker-compose up -d
```

**Redis虽然扩展比较容易，但是这里先以最简单的来，也是compose启动 **
启动Cache服务
```shell
root@blue-db-master:~# cd /var/project/blue/cache/
root@blue-db-master:/var/project/blue/data# docker-compose up -d
```

## Web/APP部署
nginx每个节点跑一个，app也是一样
```shell
root@blue-db-master:~# docker stack deploy --compose-file /var/project/blue/app/docker-compose.yml blue
root@blue-db-master:~# docker service scale blue_lb=2
root@blue-db-master:~# docker service scale blue_lb=2
```

检查日志
```docker
root@blue-db-master:~# docker service logs -f blue_web
root@blue-db-master:~# docker service logs -f blue_lb 
```

## 配置SLB
略过


## 配置DNS解析
略过