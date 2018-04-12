# nginx 调用lua

## 示例
**hello world**
```nginx
    location /hello {
        default_type 'text/plain';
        content_by_lua 'ngx.say("hello world!")';
    }
```


**获取nginx内置变量**
```nginx
    location /myip {
        default_type 'text/plain';
        content_by_lua '
             ngx.say("romote_addr: ", ngx.var.remote_addr)
        ';
    }
```


**获取所有请求头**
```nginx
    location /myip {
        default_type 'text/plain';
        content_by_lua '
             client_request_headers = ngx.req.get_headers()
                 for k, v in pairs(client_request_headers) do
                     ngx.say(k, ": ", v)
                 end
        ';
    }
```

访问结果
```
host: www.gray_release.com
accept-language: zh-CN,zh;q=0.8
accept-encoding: gzip, deflate, sdch
connection: keep-alive
accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
user-agent: Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36
upgrade-insecure-requests: 1
cache-control: max-age=0
```

**获取请求头中某个属性**
```
    location /myip {
        default_type 'text/plain';
        content_by_lua '
             -- client_request_host = ngx.req.get_headers().host
             client_request_host = ngx.req.get_headers()["host"]
             ngx.say("Host: ", client_request_host)

        ';
    }
```