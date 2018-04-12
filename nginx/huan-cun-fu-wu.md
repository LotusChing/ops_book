# 代理缓存服务

* 客户端缓存：浏览器缓存，expire、etag、last-modified
* 代理缓存：nginx
* 服务端缓存：memcached、redis


## 代理缓存参数
配置示例
```bash
    upstream flask_with_cache {
        server 192.168.2.20:5001;
        server 192.168.2.20:5002;
        server 192.168.2.20:5003;
    }
    # 缓存定义
    # 首先是缓存的目录
    # levels是缓存目录层级结构
    # key_zone定义缓存对象名称、10m为总的key的大小、通常1m~=8000key
    # max_size为最大缓存目录大小，当满了后nginx就会触发自己淘汰规则，将不常用的key淘汰掉
    # inactive定义了缓存超过60分钟没有使用就淘汰掉
    # use_temp_path定义为关闭临时缓存，不然还需要配置一个缓存的临时目录，通常会带来性能损耗
    proxy_cache_path /opt/app/cache levels=1:2 keys_zone=flask_cache:10m max_size=10g inactive=60m use_temp_path=off;

server {
	listen         80;
        server_name    www.flask.com;
        sendfile       on;
        location / {
            proxy_cache flask_cache;
            # 设置缓存过期时间，200和304为12小时
            proxy_cache_valid 200 304 12h;
            # 除了200和304其他的都是10m过期
            proxy_cache_valid any 10m;
            # 缓存hash的key
            proxy_cache_key $scheme$host$uri$is_args$args;
            # 返回给客户端是否命中
            add_header Nginx-Cache "$upstream_cache_status";
            # 当后端server返回以下异常时，访问另一个后端server
            proxy_next_upstream error timeout http_500 http_502 http_503 http_504;

            proxy_pass http://flask_with_cache/;
            include proxy_params;
        }
}
```


测试请求，可以看到头部信息中，Nginx-Cache第一次时MISS，说明没有缓存，第二次就是HIT，说明命中缓存了
```
[root@bj-vmware-test1 conf.d]# curl -I http://www.flask.com/
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Sat, 09 Dec 2017 07:56:38 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 12
Connection: keep-alive
Nginx-Cache: MISS

[root@bj-vmware-test1 conf.d]# curl -I http://www.flask.com/
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Sat, 09 Dec 2017 07:56:39 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 12
Connection: keep-alive
Nginx-Cache: HIT
```


缓存信息就在定义的目录中
```bash
[root@bj-vmware-test1 conf.d]# ls /opt/app/cache/
9
```

### 清理nginx代理缓存
* 方法一：删除全部缓存 cd cache_path && rm -rf *
* 方法二：删除指定url缓存，nginx官方不支持，第三方模块`ngx_cache_purge`

### 部分页面不缓存
```
server {
	listen         80;
        server_name    www.flask.com;
        sendfile       on;
        if ( $request_uri ~ ^/(url3|login|register|passwod|reset)){
            set $cookie_nocache 1;
        }
        location / {
            proxy_cache flask_cache;
            # 设置缓存过期时间，200和304为12小时
            proxy_cache_valid 200 304 12h;
            # 除了200和304其他的都是10m过期
            proxy_cache_valid any 10m;
            # 缓存hash的key
            proxy_cache_key $scheme$host$uri$is_args$args;
            # 返回给客户端是否命中
            add_header Nginx-Cache "$upstream_cache_status";
            # 当后端server返回以下异常时，访问另一个后端server
            proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
            # 如果cookie_nocache 为1不进行缓存，为空就进行缓存
            #proxy_no_cache $cookie_nocache $arg_nocache$arg_comment;

            proxy_pass http://flask_with_cache/;
            include proxy_params;
        }
}
```
### 大文件分片请求

优点：大请求切割成小请求，避免一个大请求断掉重新请求的问题
缺点：slice设置过小时fd损耗过多