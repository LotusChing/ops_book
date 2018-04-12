# 静态资源服务

## 需求分析
* 类型分类：不同类型的资源匹配不通的规则
* 浏览器缓存：不通类型的资源设置合理的缓存生命周期，减少不需要的性能开销
* 防盗链/盗用：设置资源的防盗链和防盗用，通过验证http-referrer和secure_link来验证资源的正常访问请求和控制链接生命周期，避免资源被随意盗用
* 流量限制：通过流量限制避免移动端强制升级时，大量客户端将静态资源服务器带宽打满无法提供正常服务的情况
* 压缩：通过配置压缩降低网络带宽的使用情况，但是要注意压缩的资源类型，那些是压缩收益高的资源，那些是压缩收益低的资源，压缩所带来的成本和收益，预编译的考虑
* 关闭access_log: 不必要的日志进行关闭以节省IO和空间
* 硬件需求：磁盘容量、转速和读写性能


处理静态资源文件时，主要参数如下

## 网络相关
* sendfile: 减少用户空间和内核空间上下文切换及拷贝次数
* tcp_nopush: 默认为off，设置为on时，提高大文件网络传输的性能，需要sendfile为on
* tcp_nodelay: 默认为on，设置为off时，缩短网络响应时间，对于交互性要求比较高的应用需要设置为off


* 具体三个参数的原理，需要我复习下网络知识，然后才能更好的理解
http://blog.51cto.com/xiaomaimai/1557773



## 压缩相关
* gzip: 是否开启压缩，默认off，建议加在location块
* gzip_http_version：http版本，默认1.1 其实并不需要设置，但是确保万一
* gzip_comp_level：压缩级别，默认为1，级别范围1-9
* gzip_types：压缩什么类型的文件
* gzip_static: 预压缩

配置语法
```bash
        location ~ .*\.(jpg|jpeg|txt)$ {
            root  html;
            gzip  on;
            gzip_http_version 1.1;
            gzip_comp_level 1;
            gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        }
```

测试后总结下
* 图片：有时压缩的不是特别好，大概三分之一到一半左右，有时会更差，因为图片本身经过压缩
* 文本：效果非常的好，15M的日志文件，压缩后浏览器获取时只有972KB，简直不要太爽


**预压缩**
简单说就是提前压缩，用来减少nginx处理请求时压缩带来的额外cpu开销，节省服务器CPU资源，但是缺点是占用磁盘空间(如果需要留双份源文件、压缩包的话)

手动预先gzip压缩
```bash
[root@bj-vmware-test1 download]# cd /opt/app/download/
[root@bj-vmware-test1 download]# ls
1.html  1.jpg
[root@bj-vmware-test1 download]# gzip 1.jpg 
[root@bj-vmware-test1 download]# ls
1.html  1.jpg.gz
```

配置语法
```bash
    location ~ ^/download {
         gzip_static on;
         tcp_nopush on;
         root /opt/app/;
    }
```

测试访问，不需要跟上.gz后缀也能访问到图片
http://192.168.2.20/download/1.jpg



## 浏览器缓存相关
* expires: 给浏览器头部添加缓存生命周期(Expires、Last-Modified)
配置语法
```bash
location / {
    expires 24h;
    root   /usr/share/nginx/html;
    index  index.html index.htm;
}
```

![](/assets/浏览器缓存.png)


## 跨站访问
跨站访问简单来说就是，客户端请求的页面里引用了另外一个服务器的内容，这种情况下客户端浏览器会认为时不安全的，所以会禁止掉

测试页面
```bash
<html lang="en">  
<head>  
<meta charset="UTF-8" />  
<title>测试ajax和跨域访问</title>  
<script src="http://libs.baidu.com/jquery/2.1.4/jquery.min.js"></script>  
</head>  
<script type="text/javascript">  
$(document).ready(function(){  
    $.ajax({  
        type: "GET",  
        url: "http://www.ching.com/index.html",  
        success: function(data) {   
            alert("sucess!!!");  
        },  
        error: function() {  
            alert("fail!!!,请刷新再试!");  
        }  
    });  
});  
</script>  
<body>  
     <h1>测试跨域访问</h1>
</body>  
</html>  
```


访问效果，除了alert信息，浏览器console里可以看到报错信息
```bash
XMLHttpRequest cannot load http://download.hrb.aiplatform.com.cn/index.html. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://192.168.2.20' is therefore not allowed access.
```


Nginx解决跨域问题的方法，**在被引用端的Nginx上设置允许跨域规则**

这是允许一个域访问的规则，如果是多个域，http://blog.sina.com.cn/s/blog_3eba8f1c0102xki2.html，目前还看不懂这个人的配置，在学习下再补充这篇文档


如果允许所有的域的话，直接配置为*就行了

```
server {
        server_name    www.ching.com;
        sendfile       on;

        location ~ .*\.(htm|html)$ {
            if ($request_method ~* "(GET|POST)") {
                add_header "Access-Control-Allow-Origin"  "*";
                add_header "Access-Control-Allow-Methods" "GET,POST,PUT,DELETE,OPTIONS";
            }
            root  /opt/app/code/ching;
        }
}
```

## 防盗链
避免非正常用户的请求给服务器带来不必要的性能开销

### 简单实现防盗链

配置如下

```
server {
	listen         80;
        server_name    192.168.2.20 www.lotus.com;
        sendfile       on;

        location ~ .*\.(htm|html)$ {
            root  /usr/share/nginx/html;
        }
        
        location ~ .*\.(png)$ {
            valid_referers none blocked *.lotus.com server_names ~\.google\. ~\.baidu\.;
            if ($invalid_referer) {
                return 403;
            }
            root /opt/app/images;
        }	
}
```

valid_referers
  * none: 允许头部"Referer"为空
  * blocked: 来源头部不为空，但是里面的值被代理或者防火墙删除了，**简单来说就是不以http://或者https://开头的**
  * server_names: 表示当前域名
  * *.lotus.com: 通配多级域名
  * ~\.google\.: 通配Google
  

使用`curl`测试配置，`-e`参数是设置referer信息

**测试允许规则**

测试none，没有referer信息时
```bash
[root@bj-vmware-test1 html]# curl -I http://192.168.2.20/test.png
HTTP/1.1 200 OK
Server: nginx/1.12.2
...
```

测试blocked，没有http或者https
```bash
[root@bj-vmware-test1 html]# curl -e "www.test.com" -I http://192.168.2.20/test.png
HTTP/1.1 200 OK
...
```

测试通配多级域名`*.lotus.com`
```bash
[root@bj-vmware-test1 html]# curl -e "http://a1.lotus.com" -I http://192.168.2.20/test.png
HTTP/1.1 200 OK
Server: nginx/1.12.2
...

[root@bj-vmware-test1 html]# curl -e "http://a2.lotus.com" -I http://192.168.2.20/test.png
HTTP/1.1 200 OK
Server: nginx/1.12.2
...
```

测试正则匹配google、baidu
```bash
[root@bj-vmware-test1 html]# curl -e "http://img.google.com" -I http://192.168.2.20/test.png
HTTP/1.1 200 OK
Server: nginx/1.12.2
...

[root@bj-vmware-test1 html]# curl -e "http://img.baidu.com" -I http://192.168.2.20/test.png
HTTP/1.1 200 OK
Server: nginx/1.12.2
...
```


**测试允许规则之外的请求**
```bash
[root@bj-vmware-test1 html]# curl -e "http://ching.com" -I http://192.168.2.20/test.png
HTTP/1.1 403 Forbidden
Server: nginx/1.12.2
...
```
