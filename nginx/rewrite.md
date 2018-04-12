# rewrite 

## 应用场景
* seo优化，伪静态
* 降级
* 配合后端api设计


## 辅助工具 pcretest
```bash
[root@bj-vmware-test1 ~]# pcretest
PCRE version 7.8 2008-09-05

  re> /(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/
data> 192.168.1.1
 0: 192.168.1.1
 1: 192
 2: 168
 3: 1
 4: 1
```

## rewrite 语法

作用域：server、location、if
语法：rewrite 匹配规则 重写后地址 flag

### flag
* break: 停止rewrite检测
* last: 停止rewrite检测
* redirect: 返回302临时重定向，浏览器会变更重写后的地址
* permanent: 返回301永久重定向，浏览器变更重写后的地址


#### last、break区别
**Example 1: No (break or last) flags**

首先定义了三个location，`/`、`/notes`、`/documents`这三个location定义返回了一些字符串

```
    location / {
        return 200 'finally matched location /\n';
    }

    location /notes {
        return 200 'finally matched location /notes\n';
    }

    location /documents {
        return 200 'finally matched location /documents\n';
    }

    rewrite ^/([^/]+.txt)$ /notes/$1;
    rewrite ^/notes/([^/]+.txt)$ /documents/$1;
```

其次这时rewrite规则是写在server段，并且没有设置flag的

请求下/test.txt看下返回什么
```
[root@bj-vmware-test1 test]# curl www.lotus.com/test.txt
finally matched location /documents
```

这说明了什么？
1. 匹配到了第一条rewrite规则，`/test.txt`请求被重写成`/notes/test.txt`
2. 重写后又匹配到了第二条规则，`/notes/test.txt`又被重写为`/documents/test.txt`
3. `location /documents`匹配到了重写后的地址，于是return相应字符串


**Example 2: Outside location block (break or last)**
这里将第一条rewrite规则加上break或者last标识
```
    location / {
        echo 'finally matched location /';
    }

    location /notes {
        echo 'finally matched location /notes';
    }

    location /documents {
        echo 'finally matched location /documents';
    }

    rewrite ^/([^/]+.txt)$ /notes/$1 break; # or last
    rewrite ^/notes/([^/]+.txt)$ /documents/$1; # this is not parsed
```

测试下刚才的地址，看返回什么，可以看到无论是`break`还是`last`返回的结果都是一样的，都是`location /notes`匹配并返回的，说明**last和break停止向后匹配**的特点生效了
```
[root@bj-vmware-test1 test]# curl www.lotus.com/test.txt
finally matched location /notes
[root@bj-vmware-test1 test]# vim /etc/nginx/conf.d/static_server.conf 
[root@bj-vmware-test1 test]# nginx -s reload
[root@bj-vmware-test1 test]# curl www.lotus.com/test.txt
finally matched location /notes
```

**Example 3: Inside location block - "break"**
把两条rewrite规则放进`location /`中
```
    location / {
        echo 'finally matched location /';
        rewrite ^/([^/]+.txt)$ /notes/$1 break;
        rewrite ^/notes/([^/]+.txt)$ /documents/$1; # this is not parsed
    }

    location /notes {
        echo 'finally matched location /notes';
    }

    location /documents {
        echo 'finally matched location /documents';
    }
```

按之前的理解，应该时重写到`/notes/test.txt`，但是好像并不是这样
```
[root@bj-vmware-test1 test]# curl www.lotus.com/test.txt
finally matched location /
```

难道是return后的rewrite指令都不生效了？

> [官方说明](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#return)： Stops processing and returns the specified code to a client. The non-standard code 444 closes a connection without sending a response header.

确实return后rewrite都没有执行

由于我最初是参考这个[回答](https://serverfault.com/a/829148/371328)，但是默认nginx没有echo模块，所以我上面都是使用return来代替，既然return会影响后面的指令，那echo或许不太一样呢？



于是我重新编译了一遍nginx，编译时添加了echo模块
```
[root@bj-vmware-test1 nginx-1.12.2]# nginx -V
nginx version: nginx/1.12.2
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-18) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx ... --add-module=/usr/local/src/echo-nginx-module-master/
```

修改nginx配置再试下看echo是否会和return有区别
```
    location / {
        echo 'finally matched location /';
        rewrite ^/([^/]+.txt)$ /notes/$1 break;
        rewrite ^/notes/([^/]+.txt)$ /documents/$1;
    }
```

访问测试，看起来和return差不多，无法确定后面rewrite是否执行了(感觉像是没执行)
```
[root@bj-vmware-test1 conf.d]# curl www.lotus.com/test.txt
finally matched location /
```

于是我在`location /`块中的最末位再加一个echo end，如果end能返回说明rewrite确实被执行了，echo_flush确保每条echo不被缓冲，直接返回给前端
```
    location / {
        echo 'finally matched location /';
        rewrite ^/([^/]+.txt)$ /notes/$1 break;
        rewrite ^/notes/([^/]+.txt)$ /documents/$1;
        echo end;
        echo_flush;
    }
```

测试请求，看到end了，说明rewrite确实执行了
```
[root@bj-vmware-test1 conf.d]# curl www.lotus.com/test.txt
finally matched location /
end
```




**Example 4: Inside location block - "last"**
为了确定下，我把break改成了last，验证了上面的想法，`/test.txt`成功重写到了`/notes/test.txt`

```
    location / {
        echo 'finally matched location /';
        rewrite ^/([^/]+.txt)$ /notes/$1 last;
        rewrite ^/notes/([^/]+.txt)$ /documents/$1;
        echo end;
        echo_flush;
    }
    
[root@bj-vmware-test1 conf.d]# curl www.lotus.com/test.txt
finally matched location /notes
```


至此，算是明白一点rewrite中break和last的共性和区别了

共性：
* 如果rewrite规则匹配到了，停止后续的rewrite匹配哪怕规则符合

区别：
* break：规则匹配到后，将rewrite后的地址在当前语句块中执行(Example 2)
* last：规则匹配后，将rewrite后的地址在server块中匹配执行，同时忽略所有其他rewrite


#### redirect和permanent的区别

配置示例
```bash
    location ~ ^/baidu {
        rewrite ^/baidu http://baidu.com redirect;
        #rewrite ^/baidu http://baidu.com permanent;
    }
```

浏览器访问`www.lotus.com`，我请求了两次，都成功跳转到百度，通过日志输出可以看到两条日志记录

```log
192.168.2.99 - - [10/Dec/2017:00:20:16 +0800] "GET /baidu HTTP/1.1" 302 161 "-" "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36" "-"
192.168.2.99 - - [10/Dec/2017:00:20:45 +0800] "GET /baidu HTTP/1.1" 302 161 "-" "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36" "-"
```

修改配置为`permanent`，测试访问日志输出，无论我浏览器访问多少次，都只有一条访问日志记录
```log
192.168.2.99 - - [10/Dec/2017:00:23:48 +0800] "GET /baidu HTTP/1.1" 301 185 "-" "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36" "-"
```

总结下，redirect和permanent的区别
* redirect(302): 临时重定向，常用于web服务器处于维护或故障状态时，临时跳转至其他web服务器，客户端每次都会先请求原web服务器，等原web服务器返回302状态码后才请求跳转后地址
* permanent(301)：永久重定向，常用于切换域名时web服务器返回302通知浏览器访问新的域名，不同于302，301状态码会永久保存跳转后的地址，后续同样的请求都会直接到跳转后的地址，不会经过原web服务器


#### rewrite优先级
1. server块中rewrite指令
2. location匹配
3. location块中rewrite指令

