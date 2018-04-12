# 负载均衡服务

按地域划分
* GSLB: 全局负载均衡，国家或省为单位
* SLB: 常用负载均衡

按OSI划分
* lay4: 传输层负载均衡(性能好)
* lay7: 应用层负载均衡(功能多)

首先写了个flask小应用，作为后端server，python代码如下
```python
import sys
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def index():
    return 'Flask: {}'.format(port)

if __name__ == '__main__':
    port=int(sys.argv[1])
    app.run(host='0.0.0.0', port=port, debug=True)
```

启动多个后端实例，这里起了三个
```
[root@bj-vmware-test1 nginx]# python35 app.py 5001
```


启动完成后，配置nginx负载均衡
```bash
    upstream flask {
        server 192.168.2.20:5001;
        server 192.168.2.20:5002;
        server 192.168.2.20:5003;
    }
server {
	listen         80;
        server_name    www.flask.com;
        sendfile       on;
        location / {
            proxy_pass http://flask/;
            include proxy_params;
        }
}
```

## 调度中的状态参数
官方文档：http://nginx.org/en/docs/http/ngx_http_upstream_module.html
* weight：权重，默认rr，权重值越大，被调度的几率越高
* down：不进行调度
* backup：正常情况下不进行调度，当所有调度节点失效时才参与调度
* max_fails：允许请求失败的次数
* fail_timeout：请求失败后，恢复调度的时间，默认10s
* max_conns：限制最大接收的连接数，**后端节点性能不一致时配置**


## 调度中的算法参数
* 默认是rr
* 加权轮训weight
* ip_hash: 根据remote_addr进行hash，这样来自同一个ip客户度访问到同一个后端节点
* least_conn: 优先调度连接数最少的后端
* url_hash: 

## 测试
### 测试backup
```bash
    upstream flask {
        server 192.168.2.20:5001 down;
        server 192.168.2.20:5002 backup;
        server 192.168.2.20:5003 max_fails=1 fail_timeout=10s;
    }
```
测试访问，正常情况下应该访问到的时5003，然后使用iptable限制5003访问，测试能否访问到backup节点5002
```bash
iptables -A INPUT -p tcp --dport 5003 -j DROP
```

正常情况下，第一次请求会卡上一小会，因为nginx在检测5003是否不可用，不出意外的话，稍等一会后nginx会将请求调度到5002

### 测试weight加权轮训
upstream配置
```
    upstream flask {
        server 192.168.2.20:5001 down;
        server 192.168.2.20:5002 weight=5;
        server 192.168.2.20:5003 max_fails=1 fail_timeout=10s;
    }
```

测试结果
```bash
[root@bj-vmware-test1 conf.d]# for i in `seq 10`; do echo `curl -s http://www.flask.com/`; done
Flask: 5002
Flask: 5003
Flask: 5002
Flask: 5002
Flask: 5002
Flask: 5002
Flask: 5002
Flask: 5003
Flask: 5002
Flask: 5002
```

## 测试ip_hash源哈希

upstream配置
```bash
    upstream flask {
        ip_hash;
        server 192.168.2.20:5001 down;
        server 192.168.2.20:5002;
        server 192.168.2.20:5003 max_fails=1 fail_timeout=10s;
    }
```

测试结果
```
[root@bj-vmware-test1 conf.d]# for i in `seq 10`; do echo `curl -s http://www.flask.com/`; done
Flask: 5003
Flask: 5003
Flask: 5003
Flask: 5003
Flask: 5003
Flask: 5003
Flask: 5003
Flask: 5003
Flask: 5003
Flask: 5003
```

源哈希问题在于，ip哈希的实现原理是通过remote_addr参数来进行hash调度，如果lb的上层还有一个反向代理的话，会反向一个现象，所有的请求都调度到一个节点

**测试时我使用了三个节点测试访问，结果都调度到了同一个节点5003**


### 测试url_hash
配置语法
```
    upstream flask {
        hash $request_uri;
        server 192.168.2.20:5001;
        server 192.168.2.20:5002;
        server 192.168.2.20:5003;
    }
server {
	listen         80;
        server_name    www.flask.com;
        sendfile       on;
        location / {
            proxy_pass http://flask/;
            include proxy_params;
        }
}
```

修改下flask脚本
```
import sys
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def index():
    return 'Flask: {}\n'.format(port)

@app.route('/url1')
def url1():
    return 'flask: {} url: 1...\n'.format(port)

@app.route('/url2')
def url2():
    return 'flask: {} url: 2...\n'.format(port)

@app.route('/url3')
def url3():
    return 'flask: {} url: 3...\n'.format(port)

@app.route('/url4')
def url4():
    return 'flask: {} url: 4 data: {}\n'.format(port, request.args.get('data'))

if __name__ == '__main__':
    port=int(sys.argv[1])
    app.run(host='0.0.0.0', port=port, debug=True)
```

flask会热更新，所以直接测试url哈希调度
```bash
[root@bj-vmware-test1 conf.d]# curl http://www.flask.com/
Flask: 5003
[root@bj-vmware-test1 conf.d]# curl http://www.flask.com/url1
flask: 5002 url: 1...
[root@bj-vmware-test1 conf.d]# curl http://www.flask.com/url2
flask: 5002 url: 2...
[root@bj-vmware-test1 conf.d]# curl http://www.flask.com/url3
flask: 5003 url: 3...


[root@bj-vmware-test1 conf.d]# curl http://www.flask.com/
Flask: 5003
[root@bj-vmware-test1 conf.d]# curl http://www.flask.com/url1
flask: 5002 url: 1...
[root@bj-vmware-test1 conf.d]# curl http://www.flask.com/url2
flask: 5002 url: 2...
[root@bj-vmware-test1 conf.d]# curl http://www.flask.com/url3
flask: 5003 url: 3...
```

但是url哈希也存在一些问题，实际生产环境中有时会通过GET方式传参，例如`http://www.flask.com/url1?username=xxx&action=xxx%timestamp=xxx`，这种情况


模拟GET传值导致url哈希无效问题
```bash
[root@bj-vmware-test1 conf.d]# curl www.flask.com/url4?username="asd"
flask: 5002 url: 4 username: asd
[root@bj-vmware-test1 conf.d]# curl www.flask.com/url4?username="qwe"
flask: 5003 url: 4 username: qwe
[root@bj-vmware-test1 conf.d]# curl www.flask.com/url4?username="zxc"
flask: 5002 url: 4 username: zxc
[root@bj-vmware-test1 conf.d]# curl www.flask.com/url4?username="qaz"
flask: 5003 url: 4 username: qaz
[root@bj-vmware-test1 conf.d]# curl www.flask.com/url4?username="wsx"
flask: 5002 url: 4 username: wsx
```

解决思路：
1. 首先获取作为hash的关键字key的值，例如username，然后拼接url+关键字值作为hash_key，利用这个hash_key进行调度
