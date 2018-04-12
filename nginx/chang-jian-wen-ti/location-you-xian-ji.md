# location 优先级


匹配符
* `=`: 精确匹配，1th，匹配到后不向后匹配
* `^~`: 前缀匹配，2th，匹配到后不向后匹配
* `~`: 正则匹配，匹配到后向后匹配，如果有更精确的匹配更精确的location，区分大小写
* `~*`:正则匹配，匹配到后向后匹配，如果有更精确的匹配更精确的location，不区分大小写

测试优先级配置
```
        #location = /code/ {
        #    rewrite ^(.*)$ /code1/index.html break;
        #}
        
        location ~ /code.* {
            rewrite ^(.*)$ /code3/index.html break;
        }

        #location ^~ /code/ {
        #    rewrite ^(.*)$ /code2/index.html break;
        #}

```


补充说明
* location @name: 命名location，用于内部服务跳转，例如lua中ngx.exec('@name')，意为跳转到指定的location


例如
```python
        location /img/ {
            root         /data/images/;
            # /data/images/img/$request_filename
            error_page 404 @img_err;
        }

        location @img_err {
           default_type text/plain;
           return 200 'Image Not Found.';
        }
```

测试效果
```
[root@Da nginx]# curl -I http://192.168.2.30/img/dog.jpg
HTTP/1.1 200 OK
Server: nginx/1.10.3
Date: Tue, 09 Jan 2018 15:02:09 GMT
Content-Type: image/jpeg
Content-Length: 180362
Last-Modified: Wed, 13 Dec 2017 00:05:30 GMT
Connection: keep-alive
ETag: "5a306eca-2c08a"
Accept-Ranges: bytes

[root@Da nginx]# curl -I http://192.168.2.30/img/dog.jpg1
HTTP/1.1 404 Not Found
Server: nginx/1.10.3
Date: Tue, 09 Jan 2018 15:02:11 GMT
Content-Type: text/plain
Content-Length: 16
Connection: keep-alive
```

## 常用location示例
### 指定多个项目配置upstream
```nginx
location ~ ^/(project1|project2) {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://project_backends;
}
```