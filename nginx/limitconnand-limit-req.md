# limit_conn & limit_req

连接请求限制：
* 连接：经过三次握手的TCP连接
* 请求：三次握手后的http请求，换言之HTTP请求建立在TCP连接上
* HTTP 1.0：一个连接对应一个请求
* HTTP 1.1：基于一个连接可以多次http请求，顺序性TCP复用
* HTTP 2.0：多路复用TCP复用？



配置语法
```bash
http {
	limit_conn_zone $binary_remote_addr zone=conn_zone:1m;
	# $binary_remote_addr比$remote_addr要小10个字节，一个会话就省10个字节，假如会话多了的话。。。那省的内存就不谈了的嘛~
	limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s;
}

location / {
	root   /usr/share/nginx/html;
	### 开启连接限制，同一时刻只允许一个IP一个连接
	limit_conn conn_zone 1;
	### 开启请求限制，每秒钟只允许一个请求，但是延迟响应3个请求 
	#limit_req zone=req_zone burst=3 nodelay;
	### 开启请求限制，每秒钟只允许一个请求，其他请求返回503
	#limit_req zone=req_zone;
	index  index.html index.htm;
}
```

PS：由于多个请求可以建立在一个连接上的特点，限制请求比限制连接更有效果

