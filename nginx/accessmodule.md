# access_module

配置语法
```bash
#    location ~ /admin.html {
#        root   /opt/app/code;
#        deny 192.168.2.99;
#        allow all;
#        index  index.html index.htm;
#    }

	location ~ /admin.html {
		root   /opt/app/code;
		#allow 192.168.2.99;
		allow 192.168.2.0/24;
		deny all;
		index  index.html index.htm;
	}
```

局限性：access_module 访问控制是根据remote_addr中的ip来进行控制的，如果使用黑名单的话，客户端用代理的话就逃避了黑名单

解决方法：
* 思路一：通过HTTP_X_FORWARDED_FOR来进行访问控制，但是也存在一些问题，就是HTTP_X_FORWARDED_FOR可以被中间节点更改，而且也不是所有厂商都会按照这个规则走
* 思路二：结合Geo模块
* 思路三：自定义变量传递