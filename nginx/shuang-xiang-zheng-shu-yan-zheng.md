# 双向证书验证

 对于一般公开的接口或资源使用单向证书加密就足够了，但是对于严格要求请求来源的项目，使用双向证书加密更为安全
 

基础环境：
* centos
* nginx
* python
* flask


## 1. 安装基础环境(跳过)

## 2. 获取服务端证书
* 方式一：自己生成
* 方式二：七牛云、阿里云、Let's Encrypt
  
由于测试客户端信任程度如何，所以我在七牛云和Let's Encrypt都申请了www.lotusching.top的证书，最终结果都是信任的，Nice~

七牛云的申请过程不再记录，跟着官方文档一步步就完事了

Let's Encrypt的简单记录下

**Ubuntu 14.04**
安装certbot
```bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx
```

获取证书
```
$ sudo certbot --authenticator standalone --installer nginx --pre-hook "nginx -s stop" --post-hook "nginx"
```

将来如果到期了，可以免费续期
```
$ sudo certbot renew --dry-run
```


## 3. nginx使用ssl证书
server的配置
```python
    server {
        ssl on;
        listen         80;
        listen         443 ssl;
        ### Let's Encrypt Certificate
        #ssl_certificate /opt/nginx/conf/ssl/fullchain1.pem;
        #ssl_certificate_key /opt/nginx/conf/ssl/privkey1.pem;
        ### Qiniu Certificate
        ssl_certificate /opt/nginx/conf/ssl/ssl.crt;
        ssl_certificate_key /opt/nginx/conf/ssl/ssl.key;
        access_log     logs/access_www.log;
        server_name    www.lotusching.top;
        location / {
            proxy_pass http://127.0.0.1:5000;
        }

    }
```

确认无误后，检查语法，启动nginx或重载配置
```
nginx -t
nginx -s reload
```

## 4. flask后端模拟接口
通过python的flask框架写了个测试的接口
```python
[root@Da ssl]# cat /tmp/app.py 
import json
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return json.dumps({'code': 1, 'msg': 'ok'})


if __name__ == '__main__':
    app.run()
```

运行起来，测试下
```python
[root@Da ssl]# curl 127.0.0.1:5000
{"code": 1, "msg": "ok"}
```

通过curl测试https访问接口
```python
[root@Da ssl]# curl https://www.lotusching.top
{"code": 1, "msg": "ok"}
```

通过python的http客户端库来测试访问
```python
In [14]: resp = requests.get('https://www.lotusching.top')
In [15]: resp.content
Out[15]: b'{"code": 1, "msg": "ok"}'
```

## 5. 生成CA及客户端私钥、证书
这里为什么这里要用自己做的证书呢，我主要考虑到两个原因
1. 如果客户端多的时候，申请证书麻烦，不好做自动化(而且好像阿里云和七牛云应该都有数量限制吧)
2. 阿里云(1年)、七牛云(1年)、Let's Encrypt(3月)，证书有效期太短了，频繁更换客户端证书也很烦

所以综合考虑这里自己做CA签发证书

首先生成CA私钥和证书
```python
[root@Da ssl]# mkdir ~/private_ca_ssl && cd !$

[root@Da ssl]# openssl genrsa -out ca.key 2048
[root@Da ssl]# openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
```

然后生成客户端的私钥和证书
```python
# 创建私钥
openssl genrsa -out client.pem 1024
openssl rsa -in client.pem -out client.key

# 生成证书请求
openssl req -new -key client.pem -out client.csr

# 签发证书
openssl x509 -req -sha256 -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 3650 -out client.crt
# 校验证书是否正确
openssl verify -CAfile ca.crt client.crt

# 如果给浏览器使用的话，需要生成一份给浏览器导入的
# 制作p12证书(导入浏览器)
openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12
```

## 6. 测试双向证书验证
修改nginx配置，加入`ssl_client_certificate`和`ssl_verify_client`，ssl_client_certificate是刚刚生成的ca证书的位置，我将ca.crt拷贝到了nginx配置目录下的ssl目录下
```python
    server {
        ssl on;
        listen         80;
        listen         443 ssl;
        #ssl_certificate /opt/nginx/conf/ssl/fullchain1.pem;
        #ssl_certificate_key /opt/nginx/conf/ssl/privkey1.pem;
        ssl_certificate /opt/nginx/conf/ssl/ssl.crt;
        ssl_certificate_key /opt/nginx/conf/ssl/ssl.key;
        ssl_client_certificate /opt/nginx/conf/ssl/ca.crt;
        ssl_verify_client on;

        access_log     logs/access_www.log;
        server_name    www.lotusching.top;
        location / {
            proxy_pass http://127.0.0.1:5000;
        }

    }
```

检查语法配置，重载配置文件
```
nginx -t
nginx -s reload
```

通过curl测试客户端没有证书和私钥，可以看到是返回400错误，400错误常用于标识客户端的错误，例如缺少参数等等，比如阿里云各项目的SDK，如果缺少请求参数就会返回400和缺少的字段信息，跑题了
```python
curl https://www.lotusching.top
<html>
<head><title>400 No required SSL certificate was sent</title></head>
<body bgcolor="white">
<center><h1>400 Bad Request</h1></center>
<center>No required SSL certificate was sent</center>
<hr><center>nginx/1.10.3</center>
</body>
</html>
```

通过python的http客户端库requests测试，错误是一样的
```python
In [16]: resp = requests.get('https://www.lotusching.top')

In [17]: resp.content
Out[17]: b'<html>\r\n<head><title>400 No required SSL certificate was sent</titl
e></head>\r\n<body bgcolor="white">\r\n<center><h1>400 Bad Request</h1></center>
\r\n<center>No required SSL certificate was sent</center>\r\n<hr><center>nginx/1
.10.3</center>\r\n</body>\r\n</html>\r\n'
```


那我这里请求时加上证书和私钥再用curl测试下
```python
curl --key client.key --cert client.crt -XGET https://www.lotusc
hing.top
{"code": 1, "msg": "ok"}
```

通过python再测试下
```python
In [18]: resp = requests.get('https://www.lotusching.top', cert=('client.crt', '
client.key'))

In [19]: resp.content
Out[19]: b'{"code": 1, "msg": "ok"}'
```



到此配置就算完了，最后附上参考的地址
* https://mp.weixin.qq.com/s/uzq9nt2Lsnp9LwMAJzQUXg
* https://letsencrypt.org/getting-started/

## 7. 补充浏览器导入证书

生成client的p12格式证书，我这里没有输入密码
```python
[root@Da private_ca_ssl]# openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12
Enter Export Password:
Verifying - Enter Export Password:
```

导入浏览器，以Chrome为例
设置 > 高级 > 管理证书 > 个人 > 导入 > 选择p12证书


关闭浏览器重新，如果访问站点还是400，关闭浏览器重新打开，不出意外就会看到类似这样的界面
![](/assets/twice-ssl-1.png)

点击确定后就可以看到正确的内容了
![](/assets/twicessl-2.png)