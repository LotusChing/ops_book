# try_files

类似py中的异常处理

配置语法
```
server {
	listen         80;
        server_name    192.168.2.20 www.lotus.com;
        sendfile       on;
        root           /opt/app/code;
        location / {
            try_files $uri @flask_backup;
        }

        location @flask_backup {
            proxy_pass http://127.0.0.1:5001;
        }

}
```
flask脚本内容
```
import sys
from flask import Flask, request
app = Flask(__name__)

@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def catch_all(path):
    return 'You want path: %s, But not exsit...' % path

if __name__ == '__main__':
    app.run()

if __name__ == '__main__':
    port = int(sys.argv[1])
    app.run(host='0.0.0.0', port=port, debug=True)
```


如果请求的文件存在，则返回文件内容，如果请求的文件不存在，则通过`@flask_backup`访问其他服务

```
[root@bj-vmware-test1 conf.d]# curl www.lotus.com/index.html
Real Server.
[root@bj-vmware-test1 conf.d]# mv /opt/app/code/index.html{,.bak}
[root@bj-vmware-test1 conf.d]# curl www.lotus.com/index.html
You want path: index.html, But not exsit...
```


这里异常情况是404，不知道是否支持403、500、502、503等异常，如果支持这些的话，那就厉害了~