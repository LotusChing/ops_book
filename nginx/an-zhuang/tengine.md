# Tengine

主要安装和nginx一样，这里记下常用的编译参数，需要补充说明的是nginx和tengine有些指令并不兼容，所以如果两者替换的话，一定要仔细检查配置中的指令是否兼容，确认无误后再做切换

环境依赖
```bash
# yum install gcc gcc-c++ ncurses-devel perl cmake openssl-devel pcre-devel
```

编译参数
```
--prefix=/opt/tengine --with-http_dyups_module --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --with-http_upstream_check_module --with-http_realip_module --with-pcre --with-http_dav_module
```

