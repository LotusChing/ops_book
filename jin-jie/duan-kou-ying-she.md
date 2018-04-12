# 端口映射

## 映射所有/指定地址的指定端口到容器的指定端口
**所有**
```docker
# docker run -d -p 5000:5000 --name flask_pro1 flask_project_1
09072ef20641b52cef8a18426103c1745e31c10a97913f67b1fcf9436ddfae79

# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                              NAMES
09072ef20641        flask_project_1     "python app.py"     34 seconds ago      Up 33 seconds       0.0.0.0:5000->5000/tcp, 5001/tcp   flask_pro1

# curl http://127.0.0.1:5000/
Hi, Flask
```

**指定**
```docker
# docker run -d -p 10.169.93.103:5000:5000 --name flask_pro1 flask_project_1
eac56234c4c1e13a88ed64bba81218128686e83abd843d238f8c26bd89460f6e

# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                    NAMES
eac56234c4c1        flask_project_1     "python app.py"     52 seconds ago      Up 51 seconds       10.169.93.103:5000->5000/tcp, 5001/tcp   flask_pro1

# curl http://127.0.0.1:5000
curl: (7) Failed to connect to 127.0.0.1 port 5000: Connection refused
```

## 映射所有地址的随机端口到容器EXPOSE端口

```docker
# docker run -d -P --name flask_pro1 flask_project_1
b58188217bccb936c1148f60de47722ee17f5dffa836a12e5cb9c34cf0752354
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                            NAMES
b58188217bcc        flask_project_1     "python app.py"     2 seconds ago       Up 1 seconds        0.0.0.0:1026->5000/tcp, 0.0.0.0:1025->5001/tcp   flask_pro1

# curl http://10.169.93.103:5000
Hi, Flask
```

## 映射指定地址的随机端口到容器指定端口
```docker
# docker run -d -p 10.169.93.103::5000 --name flask_pro1 flask_project_1
2ccc3b346e65eb7f516fb9a60b56291f58f0f94652afd05598add59206f0332a

# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                    NAMES
2ccc3b346e65        flask_project_1     "python app.py"     1 seconds ago       Up 1 seconds        5001/tcp, 10.169.93.103:1024->5000/tcp   flask_pro1
69c4d2b9df97        zabbix:wecan        "bash"              4 months ago        Up 3 months         80/tcp, 10051/tcp                        zabbix_wecan

# curl 10.169.93.103:1024
Hi, Flask
```

## 查看容器端口映射配置
```docker
# docker port flask_pro1 
5000/tcp -> 10.169.93.103:1024
```



