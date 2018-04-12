# 灰度发布

灰度发布的简单来说就是让小部分用户或指定用户访问指定的环境，减少版本发布中如果出现问题，导致大面积用户收到影响

实现手段
* IP：nginx中通过$remote_addr也可以做到上面的效果，但是通过这种方式粒度太大了，因为可能很多用户都是通过一个公网出口，例如办公环境
* cookie：


## 方案一：nginx + lua + memcahed

### 环境描述
基础环境
* **OS**: CentOS release 6.6
* **Nginx**: nginx/1.12.2 (src)

Lua相关
* **lua**: lua-5.1.4-4.1.el6.x86_64(yum)
* **LuaJIT**: LuaJIT-2.0.2(src)
* **lua-nginx**: lua-nginx-module-0.10.9rc7
* **lua-resty-memcached**：lua-resty-memcached-0.11
* **memcached**: memcached-1.4.4-5.el6.x86_64(yum)

模拟后端服务
* **python**: Python 3.5.1
* **flask**: flask-0.12.2


### 预计效果
通过在memcached中配置特定IP访问灰度环境

### 效果演示
未配置IP访问灰度
```
[root@bj-vmware-test1 ~]# curl http://www.gray_release.com/
Flask: 5001
```

配置IP信息到memcached
```
[root@bj-vmware-test1 ~]# telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.

set 192.168.2.20 0 0 1
1
STORED
```

再次测试访问
```
[root@bj-vmware-test1 ~]# curl http://www.gray_release.com/
Flask: 5002
```


### 下载相关包
相关包都下载到/usr/local/src/
```
cd /usr/local/src/
wget http://nginx.org/download/nginx-1.12.2.tar.gz
wget http://luajit.org/download/LuaJIT-2.0.2.tar.gz
wget https://github.com/simpl/ngx_devel_kit/archive/v0.3.0.tar.gz
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.9rc7.tar.gz
wget https://github.com/agentzh/lua-resty-memcached/archive/v0.11.tar.gz
```

然后解压
```
tar xf nginx-1.12.2.tar.gz
tar xf LuaJIT-2.0.2.tar.gz
tar xf v0.3.0.tar.gz
tar xf lua-nginx-module-0.10.9rc7.tar.gz
tar xf v0.11.tar.gz
```

### 安装
**安装lua和memcached**
```
yum -y install lua memcached
```


**编译nginx添加lua模块**
```
cd /usr/local/src/nginx-1.12.2/
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --add-module=/usr/local/src/echo-nginx-module-master/ --add-module=/usr/local/src/ngx_devel_kit-0.3.0 --add-module=/usr/local/src/lua-nginx-module-0.10.9rc7

make && make install
```

检查编译参数`nginx -V`


**测试nginx+lua化境**
编辑配置文件，启动nginx，测试访问，PS: 域名是我写死到hosts里的
```
[root@bj-vmware-test1 src]# cat /etc/nginx/conf.d/gray_released.conf 
server {
    listen     80;
    server_name www.gray_release.com;
 
    location /hello {
        default_type 'text/plain';
        content_by_lua 'ngx.say("hello world!")';
    }
}
```

访问结果，看到hello world就算了环境ok了~
```
[root@bj-vmware-test1 src]# curl www.gray_release.com/hello
hello world!
```

### 后端服务准备
写了个小flask应用以模拟后端
```python
import sys
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def index():
    return 'Flask: {}\n'.format(port)

if __name__ == '__main__':
    port=int(sys.argv[1])
    app.run(host='0.0.0.0', port=port, debug=True)
```

启动两个flask实例，分别监听5001、5002
```
[root@bj-vmware-test1 tmp]# python35 app.py 5001
 * Running on http://0.0.0.0:5001/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 326-390-028
```

新建窗口
```
[root@bj-vmware-test1 tmp]# python35 app.py 5002
 * Running on http://0.0.0.0:5002/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 326-390-028
```

**测试访问flask**
```bash
[root@bj-vmware-test1 ~]# curl 192.168.2.20:5001
Flask: 5001
[root@bj-vmware-test1 ~]# curl 192.168.2.20:5002
Flask: 5002
```

### 启动memcached
```
/etc/init.d/memcached start
```

### 实现灰度lua脚本
nginx中gray_release.conf引用lua脚本
```
    location / {
        default_type "text/html";
        content_by_lua_file /opt/app/lua/dep.lua;
    }

    location @server {
        proxy_pass http://192.168.2.20:5001;
    }

    location @server_test {
        proxy_pass http://192.168.2.20:5002;
    }
```

**/opt/app/lua/dep.lua**
```
[root@bj-vmware-test1 src]# cat /opt/app/lua/dep.lua
-- 从请求头中获取X-Real-IP，这个变量是需要和上层约定好的，默认情况下请求头中没有它
clientIP = ngx.req.get_headers()["X-Real-IP"]
if clientIP == nil then
    -- 如果没有X-Real-IP则获取x_forwarded_for，这里其实有些疑问，据我所知x_forwarded_for的格式是 "client_ip, proxy1_ip, proxy2_ip"，如果是这样的话，在通过配置IP为key的灰度发布的思路下，通过xff应该会存在问题，嗯，有空验证下~
    clientIP = ngx.req.get_headers()["x_forwarded_for"]
end
-- 如果xff也没有获取到的话，就干脆从nginx内置变量中获取remote_addr
if clientIP == nil then
    clientIP = ngx.var.remote_addr
end
    -- 引入memcached模块，我理解呢跟python中import module一个意思
    local memcached = require "resty.memcached"
    -- 调用memcached模块的new()方法，实例化出memc对象，memc, err很像python中拆包(解包)
    local memc, err = memcached:new()
    -- 如果没有memc对象，说明实例化过程中出现问题，输出错误信息到前端
    if not memc then
        ngx.say("failed to instantiate memc: ", err)
        -- 这里的return暂时不还不清楚啥意思
        return
    end
    -- 调用memc对象的connect方法连接memcached
    local ok, err = memc:connect("127.0.0.1", 11211)
    -- 这里ok是什么也还不知道
    if not ok then
        ngx.say("failed to connect: ", err)
        return
    end
    -- 调用memc对象的get方法获取key为clientIP的value，res就是valu，flags为设置key时的flags
    local res, flags, err = memc:get(clientIP)
    if  res == "1" then
        -- 如果等于1则走location @server_test灰度环境
        result = ngx.exec("@server_test")
    end
    -- 否则走location @server正式环境
    ngx.exec("@server")
```
# 灰度发布

灰度发布的简单来说就是让小部分用户或指定用户访问指定的环境，减少版本发布中如果出现问题，导致大面积用户收到影响

实现手段
* IP：nginx中通过$remote_addr也可以做到上面的效果，但是通过这种方式粒度太大了，因为可能很多用户都是通过一个公网出口，例如办公环境
* cookie：


## 方案一：nginx + lua + memcahed

### 环境描述
基础环境
* **OS**: CentOS release 6.6
* **Nginx**: nginx/1.12.2 (src)

Lua相关
* **lua**: lua-5.1.4-4.1.el6.x86_64(yum)
* **LuaJIT**: LuaJIT-2.0.2(src)
* **lua-nginx**: lua-nginx-module-0.10.9rc7
* **lua-resty-memcached**：lua-resty-memcached-0.11
* **memcached**: memcached-1.4.4-5.el6.x86_64(yum)

模拟后端服务
* **python**: Python 3.5.1
* **flask**: flask-0.12.2


### 预计效果
通过在memcached中配置特定IP访问灰度环境

### 效果演示
未配置IP访问灰度
```
[root@bj-vmware-test1 ~]# curl http://www.gray_release.com/
Flask: 5001
```

配置IP信息到memcached
```
[root@bj-vmware-test1 ~]# telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.

set 192.168.2.20 0 0 1
1
STORED
```

再次测试访问
```
[root@bj-vmware-test1 ~]# curl http://www.gray_release.com/
Flask: 5002
```


### 下载相关包
相关包都下载到/usr/local/src/
```
cd /usr/local/src/
wget http://nginx.org/download/nginx-1.12.2.tar.gz
wget http://luajit.org/download/LuaJIT-2.0.2.tar.gz
wget https://github.com/simpl/ngx_devel_kit/archive/v0.3.0.tar.gz
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.9rc7.tar.gz
wget https://github.com/agentzh/lua-resty-memcached/archive/v0.11.tar.gz
```

然后解压
```
tar xf nginx-1.12.2.tar.gz
tar xf LuaJIT-2.0.2.tar.gz
tar xf v0.3.0.tar.gz
tar xf lua-nginx-module-0.10.9rc7.tar.gz
tar xf v0.11.tar.gz
```

### 安装
**安装lua和memcached**
```
yum -y install lua memcached
```


**编译nginx添加lua模块**
```
cd /usr/local/src/nginx-1.12.2/
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --add-module=/usr/local/src/echo-nginx-module-master/ --add-module=/usr/local/src/ngx_devel_kit-0.3.0 --add-module=/usr/local/src/lua-nginx-module-0.10.9rc7

make && make install
```

检查编译参数`nginx -V`


**测试nginx+lua化境**
编辑配置文件，启动nginx，测试访问，PS: 域名是我写死到hosts里的
```
[root@bj-vmware-test1 src]# cat /etc/nginx/conf.d/gray_released.conf 
server {
    listen     80;
    server_name www.gray_release.com;
 
    location /hello {
        default_type 'text/plain';
        content_by_lua 'ngx.say("hello world!")';
    }
}
```

访问结果，看到hello world就算了环境ok了~
```
[root@bj-vmware-test1 src]# curl www.gray_release.com/hello
hello world!
```

### 后端服务准备
写了个小flask应用以模拟后端
```python
import sys
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def index():
    return 'Flask: {}\n'.format(port)

if __name__ == '__main__':
    port=int(sys.argv[1])
    app.run(host='0.0.0.0', port=port, debug=True)
```

启动两个flask实例，分别监听5001、5002
```
[root@bj-vmware-test1 tmp]# python35 app.py 5001
 * Running on http://0.0.0.0:5001/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 326-390-028
```

新建窗口
```
[root@bj-vmware-test1 tmp]# python35 app.py 5002
 * Running on http://0.0.0.0:5002/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 326-390-028
```

**测试访问flask**
```bash
[root@bj-vmware-test1 ~]# curl 192.168.2.20:5001
Flask: 5001
[root@bj-vmware-test1 ~]# curl 192.168.2.20:5002
Flask: 5002
```

### 启动memcached
```
/etc/init.d/memcached start
```

### 实现灰度lua脚本
nginx中gray_release.conf引用lua脚本
```
    location / {
        default_type "text/html";
        content_by_lua_file /opt/app/lua/dep.lua;
    }

    location @server {
        proxy_pass http://192.168.2.20:5001;
    }

    location @server_test {
        proxy_pass http://192.168.2.20:5002;
    }
```

**/opt/app/lua/dep.lua**
```
[root@bj-vmware-test1 src]# cat /opt/app/lua/dep.lua
-- 从请求头中获取X-Real-IP，这个变量是需要和上层约定好的，默认情况下请求头中没有它
clientIP = ngx.req.get_headers()["X-Real-IP"]
if clientIP == nil then
    -- 如果没有X-Real-IP则获取x_forwarded_for，这里其实有些疑问，据我所知x_forwarded_for的格式是 "client_ip, proxy1_ip, proxy2_ip"，如果是这样的话，在通过配置IP为key的灰度发布的思路下，通过xff应该会存在问题，嗯，有空验证下~
    clientIP = ngx.req.get_headers()["x_forwarded_for"]
end
-- 如果xff也没有获取到的话，就干脆从nginx内置变量中获取remote_addr
if clientIP == nil then
    clientIP = ngx.var.remote_addr
end
    -- 引入memcached模块，我理解呢跟python中import module一个意思
    local memcached = require "resty.memcached"
    -- 调用memcached模块的new()方法，实例化出memc对象，memc, err很像python中拆包(解包)
    local memc, err = memcached:new()
    -- 如果没有memc对象，说明实例化过程中出现问题，输出错误信息到前端
    if not memc then
        ngx.say("failed to instantiate memc: ", err)
        -- 这里的return暂时不还不清楚啥意思
        return
    end
    -- 调用memc对象的connect方法连接memcached
    local ok, err = memc:connect("127.0.0.1", 11211)
    -- 这里ok是什么也还不知道
    if not ok then
        ngx.say("failed to connect: ", err)
        return
    end
    -- 调用memc对象的get方法获取key为clientIP的value，res就是valu，flags为设置key时的flags
    local res, flags, err = memc:get(clientIP)
    if  res == "1" then
        -- 如果等于1则走location @server_test灰度环境
        result = ngx.exec("@server_test")
    end
    -- 否则走location @server正式环境
    ngx.exec("@server")
```

**导入lua-memcached客户端lua**
```
cp -rf /usr/local/src/lua-resty-memcached-0.11/lib/resty  /usr/local/LuaJIT/share/luajit-2.0.2/
```

**启动nginx测试**
PS: 出现问题阅读/var/log/nginx/eror.log

### 总结
这个方案的优点：
* 满足灰度的基本需求了

缺点：
* 通过这种方式粒度太大了，因为可能很多用户都是通过一个公网出口，例如办公环境
* memcached的数据类型太少了，心里还是比较喜欢一个list存放灰度ip或者其他标识(device_id)