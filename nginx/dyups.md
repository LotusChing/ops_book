# dyups 模块

## 基本安装
模块地址：https://github.com/yzprofile/ngx_http_dyups_module

下载最新nginx稳定版
```bash
# wget https://nginx.org/download/nginx-1.12.2.tar.gz
# tar xf nginx-1.12.2.tar.gz
```

拉取dyups模块代码
```bash
# git clone git://github.com/yzprofile/ngx_http_dyups_module.git
```

编译，这里用的是默认参数，然后添加dyups模块
```
./configure --prefix=/usr/local/nginx --add-module=../ngx_http_dyups_module
```

配置nginx.conf
```nginx
http {
    include conf.d/upstream.conf;
    server {
        listen   127.0.0.1:8080;
        server_name web.lotusching.top;
        location / {
            set $ups dyhost;
            proxy_pass http://$ups;
        }
    }

    server {
        listen 127.0.0.1:8081;
        location / {
            # upstream管理端口
            dyups_interface;
        }
    }
}
```

配置upstream.conf
```
root@iZ947mgy3c5Z:/usr/local/nginx# cat conf/conf.d/upstream.conf
upstream dyhost {
    server 127.0.0.1:5001;
    server 127.0.0.1:5002;
}
```

启动测试
```
root@iZ947mgy3c5Z:/usr/local/nginx# ./sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

默认后端服务的flask应用
```python
import sys
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return '{}\n'.format(sys.argv[1])

if __name__ == '__main__':
    app.run(port=sys.argv[1])
```
启动两个服务实例
```python
# python app.py 5001
# python app.py 5002
```


## 功能测试

dyups模块可以动态修改已有的upstream配置，也可以动态添加新的upstream，然后客户端通过在请求头中声明使用那个upstream进行调度

这里主要测试第一种方式，第二种方式虽然好，但是需要修改代码，所以暂时不考虑

测试反向代理负载情况，可以看到是RR的
```bash
root@iZ947mgy3c5Z:/usr/local/nginx# curl web.lotusching.top:8080
5001
root@iZ947mgy3c5Z:/usr/local/nginx# curl web.lotusching.top:8080
5002
```

或者可以通过管理端口查看upstream后端server
```bash
root@iZ947mgy3c5Z:/usr/local/tengine# curl 127.0.0.1:8081/detail
dyhost
server 127.0.0.1:5001 weight=1 max_fails=1 fail_timeout=10 backup=0 down=0
server 127.0.0.1:5002 weight=1 max_fails=1 fail_timeout=10 backup=0 down=0
```

此时通过python发送http请求，删除其中一个后端server

**client.py**
```
import requests


def get_upstream_detail():
    resp = requests.get('http://127.0.0.1:8081/detail')
    res = resp.content.decode()
    data = {}
    for i in filter(None, res.split('\n\n')):
        upstream_name, upstream_servers = i.split('\n')[0], i.split('\n')[1:]
        data[upstream_name] = upstream_servers
    return data


def set_upstream_server(upstream_name, server):
    resp = requests.post('http://127.0.0.1:8081/upstream/{}'.format(upstream_name),data="{}".format(server))
    return resp.content.decode()


if __name__ == '__main__':
    upstream_data = get_upstream_detail()
    print('Default: {}'.format(upstream_data))
    dyhost_servers = upstream_data['dyhost']
    set_upstream_server('dyhost', "server 127.0.0.1:5001;")
    #set_upstream_server('dyhost', "server 127.0.0.1:5001;server 127.0.0.1:5002;")
    current_upstream_data = get_upstream_detail()
    print('Current: {}'.format(current_upstream_data))
    print('Done.')
```

执行python脚本，可以看到最终后端server只有一个
```nginx
root@iZ947mgy3c5Z:/usr/local/tengine# python3.5 1.py 
Default: {'dyhost': ['server 127.0.0.1:5001 weight=1 max_fails=1 fail_timeout=10 backup=0 down=0', 'server 127.0.0.1:5002 weight=1 max_fails=1 fail_timeout=10 backup=0 down=0']}
Current: {'dyhost': ['server 127.0.0.1:5001 weight=1 max_fails=1 fail_timeout=10 backup=0 down=0']}
Done.
```

再次尝试请求服务，所有请求调度到5001一台server上了
```nginx
root@iZ947mgy3c5Z:/usr/local/tengine# curl web.lotusching.top:8080
5001
root@iZ947mgy3c5Z:/usr/local/tengine# curl web.lotusching.top:8080
5001
root@iZ947mgy3c5Z:/usr/local/tengine# curl web.lotusching.top:8080
5001
```


通过管理接口查看upstream情况
```nginx
root@iZ947mgy3c5Z:/usr/local/tengine# curl 127.0.0.1:8081/detail
dyhost
server 127.0.0.1:5001 weight=1 max_fails=1 fail_timeout=10 backup=0 down=0
```


## 访问控制

```nginx
    server {
        listen 8082;
        location / {
            allow 127.0.0.1;
            allow 172.17.0.0/16;
            allow 192.168.1.0/24;
            deny all;
            dyups_interface;
        }
    }
```
