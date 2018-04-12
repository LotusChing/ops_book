# 快速构建Nginx+Tomcat集群


## 目录结构
```python
.
├── docker-compose.yml
├── nginx
│   ├── Dockerfile
│   ├── nginx.conf
│   └── nginx.repo
├── tomcat
│   ├── apache-tomcat-7.0.85.tar.gz
│   └── Dockerfile
└── tomcat_webapps
```

## 容器服务

### nginx
**nginx/Dockerfile**
```docker
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
EXPOSE 80
```

**nginx/nginx.conf**
```nginx
upstream tomcat_servers {
    server tomcat1:8080;
    server tomcat2:8080;
}
server {
    listen    80;
    server_name  localhost;
 
    location / {
        proxy_pass http://tomcat_servers;
    }
}
```

**nginx/nginx.repo**
```python
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/6/$basearch/
gpgcheck=0
enabled=1
```

### tomcat
**tomcat/Dockerfile**
```docker
FROM openjdk:8-jdk

# locales
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
RUN echo "export LANG=en_US.UTF-8" >> ~/.bashrc
RUN echo "export LANGUAGE=en_US.UTF-8" >> ~/.bashrc

# timezone
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
RUN echo "Asia/Chongqing" > /etc/timezone
ENV TZ Asia/Shanghai

# Download
ADD apache-tomcat-7.0.85.tar.gz /usr/local/
RUN mv /usr/local/apache-tomcat-7.0.85 /usr/local/tomcat
WORKDIR /usr/local/tomcat

# Startup CMD
CMD /usr/local/tomcat/bin/startup.sh && tail -f /usr/local/tomcat/logs/catalina.out
EXPOSE 8080
```
## docker-compose
**docker-compose.yml**
```docker
version: '3'
services:
  nginx:
    build: ./nginx
    links:
      - tomcat1
      - tomcat2
    ports:
      - "80:80"
  tomcat1:
    build: ./tomcat
    volumes:
      - "./tomcat_webapps:/usr/local/tomcat/webapps"
  tomcat2:
    build: ./tomcat
    volumes:
      - "./tomcat_webapps:/usr/local/tomcat/webapps"
```


## 启动
```python
docker-compose up --build
```

## 测试访问、查看亮哥容器的local_access日志