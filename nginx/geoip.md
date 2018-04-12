# geoip

作用：
* 国家地区访问控制
* 国家地区择近选择
* 获取客户端地区信息，用以分析

## 编译安装MaxMind 的 GeoIP 库

```bash
wget http://geolite.maxmind.com/download/geoip/api/c/GeoIP.tar.gz
tar -zxvf GeoIP.tar.gz
cd GeoIP-1.4.6
libtoolize -f
./configure
make && make install
```

## 配置LD库
```
echo '/usr/local/lib' > /etc/ld.so.conf.d/geoip.conf
ldconfig
```

## 重新编译nginx
获取之前的参数
```
nginx -V 
```

加上`--with-http_geoip_module`
```
./configure ... --with-http_geoip_module
make; make install
```

## 下载 IP 数据库
```
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz

gunzip GeoIP.dat.gz 
```

## 测试
```nginx
geoip_country /usr/local/src/GeoIP.dat;
server {
    listen   82;
    server_name geoip.lotusching.top;
    root     /opt/app/download;

    location / {
        default_type "text/plain";
        return 200 $geoip_country_code;
}
```

测试结果
```
[root@bj-vmware-test1 conf.d]# curl http://geoip.lotusching.top:82
CN
```


## 示例：限制国外用户访问
Nginx配置
```nginx
location / {
    default_type "text/plain";
    if ($geoip_country_code = CN) {
         return 200 "China.";
    }
    return 403 "!!!Access Deny!!!";
}
```

## 示例：指定区域访问指定节点
```nginx
location / {
    default_type "text/plain";
    if ($geoip_country_code = CN) {
        rewrite ^/(.*)$ http://cn.www.lotusching.top:82/$1 permanent;
    }
    return 403 "!!!Access Deny!!!";
    rewrite ^/(.*)$ http://us.www.lotusching.top:82/$1 permanent;
}
```
这里用了permanent，效果体现在客户端收到一个`301 Moved Permanently`后，后续的请求就会自动的请求这个跳转后的地址

这么做优点在于，减少主站请求压力，缺点在于主站判断策略更新，客户端无法即时的使用最新的策略


## 补充资料

### 日志输出国家地区城市
https://thecustomizewindows.com/2016/10/configure-nginx-access-log-geoip-ubuntu-16-04/

**nginx配置文件**
```nginx
root@f9a7c95dbe26:/etc/nginx# cat nginx.conf

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

load_module "modules/ngx_http_geoip_module.so";

events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"'
    #                  '"$geoip_country_name" "$geoip_country_code"'
    #                  '"$geoip_region" "$geoip_region_name"'
    #                  '"$geoip_city", "$geoip_city_country_name"';

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"'
                      '"$geoip_country_name", "$geoip_city"';
    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;
    
    # GeoIP
    geoip_country /etc/nginx/geoip/GeoIP.dat;
    geoip_city /etc/nginx/geoip/GeoLiteCity.dat;

    include /etc/nginx/conf.d/*.conf;
}
```

**日志输出**
```log
45.121.65.171 - - [22/Mar/2018:03:18:43 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36" "-""China", "Beijing"
```

