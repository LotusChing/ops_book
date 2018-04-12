# 快速构建LNMP
## 目录结构
```python
# tree -L 3 lnmp
lnmp
├── docker-compose.yml
├── mysql
│   ├── conf
│   │   └── my.cnf
│   ├── data
│   │   ├── auto.cnf
│   │   ├── ibdata1
│   │   ├── ib_logfile0
│   │   ├── ib_logfile1
│   │   ├── mysql
│   │   └── performance_schema
│   └── Dockerfile
├── nginx
│   ├── Dockerfile
│   ├── nginx.conf
│   └── nginx.repo
├── php
│   └── Dockerfile
└── wwwroot
    ├── index.html
    └── index.php
```

## 服务容器
### nginx
**nginx/Dockerfile**
```docker
# cat nginx/Dockerfile 
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

**nginx/nginx.conf**
```nginx
server {
    listen    80;
    server_name  localhost;
    root      html;
    index     index.html index.php;
 
    location / {
        root  html;
        index index.html;
    }

    location ~ \.php$ {
        root  html;
        fastcgi_pass php-cgi:9000;
        fastcgi_param SCRIPT_FILENAME /usr/share/nginx/html/$fastcgi_script_name;
        include fastcgi_params;
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

### php
```docker
FROM centos:6
MAINTAINER LotusChing

RUN cd /etc/yum.repos.d && mkdir bak && mv *.repo bak
RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
RUN yum -y install openssl-devel pcre-devel gcc gcc-c++ libxml2-devel libcurl-devel libjpeg-devel libpng-devel gd-devel php php-fpm php-mysql

# locales
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
RUN echo "export LC_ALL=en_US.UTF-8" >> ~/.bashrc
RUN echo "export LANG=en_US.UTF-8" >> ~/.bashrc
RUN echo "export LANGUAGE=en_US.UTF-8" >> ~/.bashrc

# timezone
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
RUN echo "Asia/Chongqing" > /etc/timezone
ENV TZ Asia/Shanghai

# Install
RUN sed -r -i 's/127.0.0.1:9000/9000/g' /etc/php-fpm.d/www.conf
RUN sed -r -i 's/^listen.allowed_clients/; listen.allowed_clients/g' /etc/php-fpm.d/www.conf
CMD /etc/init.d/php-fpm start && tail -F /var/log/php-fpm/www-error.log
EXPOSE 9000
```

### mysql
暂时并未用到Dockerfile

**mysql/conf/my.cnf**
```mysql
[mysqld]
user=mysql
port=3306
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
pid-file=/var/run/mysql/mysql.pid
log_error=/var/log/mysql/error.log
character_set_server = utf8mb4
max_connections = 3600
```

## docker-compose
**docker-compose.yml**
```yaml
version: '3'
services:
  nginx:
    hostname: nginx
    build: 
      context: ./nginx
      dockerfile: Dockerfile
    ports:
      - "80:80"
    links:
      - php:php-cgi
    volumes:
      - ./wwwroot:/usr/share/nginx/html
  php:
    hostname: php
    build: ./php
    links:
      - mysql:mysql-db
    volumes:
      - ./wwwroot:/usr/share/nginx/html
  mysql:
    hostname: mysql
    image: mysql:5.6
    ports:
      - "3306:3306"
    volumes:
      - ./mysql/conf:/etc/mysql/conf.d
      - ./mysql/data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: 123123
      MYSQL_USER: lotus
      MYSQL_PASSWORD: ching
```

## 启动
```
docker-compose up --build
```
