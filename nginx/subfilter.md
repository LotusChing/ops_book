# sub_filter

配置语法
```bash
location / {
	root   /usr/share/nginx/html;
	index  index.html index.htm;
	sub_filter 'nginx' 'LotusChing';
	# 默认只替换一次，如果需要替换所有，则需要配置sub_filter_once为off
	sub_filter_once off;
}
```

