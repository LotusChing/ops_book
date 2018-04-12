# 配置文件存储
## 配置文件存储

## 编辑配置文件
临时写一个配置文件
```nginx
root@node-40:~# cat ngx_simple.conf 
server {
    listen      80;
    server_name localhost;
    location / {
        add_header Content-Type "text/plain;charset=utf-8";
        return 200 "Your IP Address:$remote_addr";
    }
}
```

## 创建并应用配置文件
```docker
root@node-40:~# docker config create ngx_simple_config ngx_simple.conf
root@node-40:~# docker service create --name cfg_ngx --config source=ngx_simple_config,target=/etc/nginx/conf.d/default.conf --publish 8882:80 nginx:1.12
```

## 测试访问查看
```docker
root@node-40:~# curl http://192.168.2.40:8882/
Your IP Address:10.255.0.2
```