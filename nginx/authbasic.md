# auth_basic 

配置语法
```bash
# htpasswd -c auth_conf username
    location ~ /admin.html {
	root   /opt/app/code;
	auth_basic "!!!Auth access test, input user/passwd!!!";
	auth_basic_user_file /etc/nginx/auth_conf;
	index  index.html index.htm;
}
```

局限性：用户信息依赖文件方式，操作机械，加入企业内有多套系统时无法实现打通和联动，管理低效

解决方法：
	思路一：nginx结合lua实现高效验证
	思路二：nginx结合ldap，利用nginx-auth-ldap模块