# 常见错误码

## 404 Not found
请求的文件不存在，不多说 

## 410 Gone
请求的资源过期，secure_link中学习到的

## 413 Request entity too large
请求的文件太大了，常见于上传例如以http方式使用git，或者Cms中上传文件

调整nginx参数`client_max_body_size`

## 405 Method not allow
请求方法不允许，flask中学到的，默认路由仅支持`Get`方法，如果使用POST方式请求未开启`POST`的路由，则会收到405的返回


# 502 Bad Gateway
nginx中proxy_pass的后端无法连接
```
[root@bj-vmware-test1 ~]# curl -I www.lotus.com/asd
HTTP/1.1 502 Bad Gateway
Server: nginx/1.12.2
Date: Thu, 21 Dec 2017 16:46:10 GMT
Content-Type: text/html
Content-Length: 173
Connection: keep-alive
```

# 504 Gateway Time-out
nginx中proxy_pass的后端处理超时，nginx默认60秒，超过这个时间nginx返回504错误码
