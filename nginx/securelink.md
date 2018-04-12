# secure_link

作用：文件防盗链，传统下载方式存在容易被三方引用的问题，通过referer也并不能很好的解决，因为http客户端的header很容易篡改，通过secure_link模块可以很好的实现安全和控制下载链接生效周期

python生成下载链接
```python
import hashlib
import base64
import time

secret_download_salt = 'LotusChing'
def generate_secret_link(url):
    expire_ts = int(time.time()+300)
    secure_link = "{}{} {}".format(expire_ts, url, secret_download_salt)
    hash = hashlib.md5(secure_link.encode('utf-8')).digest()
    md5_secret = base64.urlsafe_b64encode(hash).rstrip('='.encode('utf-8'))
    return '{}?md5={}&expires={}'.format(url, md5_secret.decode(), expire_ts)

server_name="www.download.com"
url = '/test.img'
secret_download_url = generate_secret_link(url)
print("Download url: {}".format("http://" + server_name + secret_download_url))
```



配置语法
```
server {
	listen         80;
        server_name    www.download.com;
        sendfile       on;
        root           /opt/app/download;

    location / {
        secure_link $arg_md5,$arg_expires;
        secure_link_md5 "$secure_link_expires$uri LotusChing";

        if ($secure_link = "") {
            return 403;
        }

        if ($secure_link = "0") {
            return 410;
        }
    }

}
```


校验失败返回403，url过期返回410
![](/assets/secure_lin_410.png)

## 参考实例：某国外视频站
点击播放后，播放一小段广告，广告结束后，请求视频链接206部分请求，st应该就是哈希的值，e就是过期时间
`http://1.1.1.1/MEF-ZWJDeQ5dXXU=.mp4?st=7gzY5qOC6zRiDjULCVwGDQ&e=1517882096"`


这个站点除了以下几个安全手段
* hash & expire
* referrer
* 单点下载，同一IP只允许并行下载一个视频
* 下载限速