# 代理服务

正向代理和反向代理的关键区别，在于代理的对象不一样

* 正向代理代理的是客户端
* 反向代理是代理服务端

## 需求分析
* 代理协议类型：http、websocket
* 代理方向：正向还是反向
* 反向代理：是否需要负载均衡，需要什么调度策略，fail检查机制，failtimeout时间
* 头信息处理：remote_addr、自定义头信息传递给后端
* 硬件需求：CPU、内存

## 正向

简单配置语法
```bash
server {
	listen         8081;
        sendfile       on;

        location / {
            resolver 8.8.8.8;
            proxy_pass http://$http_host$request_uri;
        }
}
```

## 反向
配置语法
```bash
server {
	listen         80;
        server_name    www.proxy.com;
        sendfile       on;

        location / {
            proxy_pass http://127.0.0.1:8080/;
        }
}
```

### 关键参数
**后端地址**和**公共参数**
```bash
proxy_pass url;
include proxy_params;
```
### 公共参数
```bash
[root@bj-vmware-test1 nginx]# cat /etc/nginx/proxy_params
proxy_redirect default;
# 头信息设置，k是Host，v是请求的地址，不包括uri 
proxy_set_header Host $http_host;
# 头信息设置，k是X-Real-IP，v是远程地址
proxy_set_header X-Real-IP $remote_addr;
# 建立连接的超时时间
proxy_connect_timeout 30;
# 读取后端返回的超时时间
proxy_read_timeout 60;
# 返回后端数据到客户端的超时时间
proxy_send_timeout 60;
# 头信息缓冲区
proxy_buffer_size 32k;
# 延迟返回，等待后端返回完在返回给前端
proxy_buffering on;
# 设置返回内容缓冲大小
proxy_buffers 4 128k;
# 不清楚这个参数的具体意义
proxy_busy_buffers_size 256k;
# 缓冲区空余空间不足或者返回内容过大时，会使用临时文件，这里是配置临时文件的大小
# 临时文件存储位置，取决于默认取决于编译时的参数 --http-proxy-temp-path=/var/cache/nginx/proxy_temp
proxy_max_temp_file_size 256k;
```

### 验证X-Real-IP
临时写了个flask应用测试
```python
[root@bj-vmware-test1 tmp]# cat app.py 
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def index():
    headers = request.headers
    remote_addr = request.remote_addr
    print('### Headers ###\n{}'.format(headers))
    print('### Remote Addr: {} ###'.format(remote_addr))
    print('### X-Real-IP: {} ###'.format(headers.get('X-Real-IP')))
    return 'Da.'

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```


测试访问
```
### Headers ###
Accept-Encoding: gzip, deflate, sdch
Host: www.proxy.com
Accept-Language: zh-CN,zh;q=0.8
Connection: close
Upgrade-Insecure-Requests: 1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
X-Real-Ip: 192.168.2.99
User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36
Cache-Control: max-age=0


### Remote Addr: 192.168.2.20 ###
### X-Real-IP: 192.168.2.99 ###
192.168.2.20 - - [09/Dec/2017 12:32:01] "GET / HTTP/1.0" 200 -
```




