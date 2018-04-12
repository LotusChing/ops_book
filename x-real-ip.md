# X-Real-IP

## 问题描述

## 问题现象

## 解决方法
### Nginx配置
`proxy_set_header`指令意为给HTTP请求头加一个变量，Tomcat通过获取变量来获取数据
```nginx
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-Host $host;
proxy_set_header X-Forwarded-Server $host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```


### Tomcat配置
Tomcat引用请求头中的`X-Real-IP`变量，有很多人用的是`X-Forwarded-For`，但是我这里没用，是因为这个变量通常用来记录经过的所有代理节点的IP，所以有可能是多个，所以这里没用
```xml
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
    prefix="localhost_access_log." suffix=".txt"
    pattern="%{X-Real-IP}i %l %u %t &quot;%r&quot; %s %b &quot;%{Referer}i&quot; &quot;%{User-Agent}i&quot;" />
```