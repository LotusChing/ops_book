## 快速部署Flask+Redis
```python
# tree test1
test1
├── app.py
├── docker-compose.yml
├── Dockerfile
└── requirements.txt
```

**app.py**
```python
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def index():
    count = redis.incr('hits')
    return 'LotusChing. Hits {} times'.format(count)

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```

**requirements.txt**
```
flask
redis
```

**Dockerfile**
```python
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

**docker-compose.yml**
```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```


**启动(Up)**
默认前台启动，如果要运行在后台，`docker-compose up -d`
```bash
# docker-compose up
Creating network "test1_default" with the default driver
Building web
Step 1/5 : FROM python:3.4-alpine
 ---> 5c72717ec319
Step 2/5 : ADD . /code
 ---> dfd59aa119dc
Removing intermediate container 5334818d031e
Step 3/5 : WORKDIR /code
 ---> b532bf4ca2b3
Removing intermediate container 2bc7c745b2e5
Step 4/5 : RUN pip install -r requirements.txt
 ---> Running in 6350c358e38f
Collecting flask (from -r requirements.txt (line 1))
  Downloading Flask-0.12.2-py2.py3-none-any.whl (83kB)
Collecting redis (from -r requirements.txt (line 2))
  Downloading redis-2.10.6-py2.py3-none-any.whl (64kB)
Collecting itsdangerous>=0.21 (from flask->-r requirements.txt (line 1))
  Downloading itsdangerous-0.24.tar.gz (46kB)
Collecting click>=2.0 (from flask->-r requirements.txt (line 1))
  Downloading click-6.7-py2.py3-none-any.whl (71kB)
Collecting Jinja2>=2.4 (from flask->-r requirements.txt (line 1))
  Downloading Jinja2-2.10-py2.py3-none-any.whl (126kB)
Collecting Werkzeug>=0.7 (from flask->-r requirements.txt (line 1))
  Downloading Werkzeug-0.14.1-py2.py3-none-any.whl (322kB)
Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->flask->-r requirements.txt (line 1))
  Downloading MarkupSafe-1.0.tar.gz
Building wheels for collected packages: itsdangerous, MarkupSafe
  Running setup.py bdist_wheel for itsdangerous: started
  Running setup.py bdist_wheel for itsdangerous: finished with status 'done'
  Stored in directory: /root/.cache/pip/wheels/fc/a8/66/24d655233c757e178d45dea2de22a04c6d92766abfb741129a
  Running setup.py bdist_wheel for MarkupSafe: started
  Running setup.py bdist_wheel for MarkupSafe: finished with status 'done'
  Stored in directory: /root/.cache/pip/wheels/88/a7/30/e39a54a87bcbe25308fa3ca64e8ddc75d9b3e5afa21ee32d57
Successfully built itsdangerous MarkupSafe
Installing collected packages: itsdangerous, click, MarkupSafe, Jinja2, Werkzeug, flask, redis
Successfully installed Jinja2-2.10 MarkupSafe-1.0 Werkzeug-0.14.1 click-6.7 flask-0.12.2 itsdangerous-0.24 redis-2.10.6
 ---> 4c728f3ca13a
Removing intermediate container 6350c358e38f
Step 5/5 : CMD python app.py
 ---> Running in 26ac4d4b4f0a
 ---> 1b0dab6a6b76
Removing intermediate container 26ac4d4b4f0a
Successfully built 1b0dab6a6b76
Successfully tagged test1_web:latest
WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating test1_web_1 ... 
Creating test1_redis_1 ... 
Creating test1_web_1
Creating test1_web_1 ... done
Attaching to test1_redis_1, test1_web_1
redis_1  | 1:C 18 Feb 11:33:51.559 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 18 Feb 11:33:51.561 # Redis version=4.0.8, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 18 Feb 11:33:51.561 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis_1  | 1:M 18 Feb 11:33:51.570 * Running mode=standalone, port=6379.
redis_1  | 1:M 18 Feb 11:33:51.570 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis_1  | 1:M 18 Feb 11:33:51.570 # Server initialized
redis_1  | 1:M 18 Feb 11:33:51.571 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
redis_1  | 1:M 18 Feb 11:33:51.571 * Ready to accept connections
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web_1    |  * Restarting with stat
web_1    |  * Debugger is active!
web_1    |  * Debugger PIN: 620-220-430
web_1    | 192.168.2.99 - - [18/Feb/2018 11:34:23] "GET / HTTP/1.1" 200 -
web_1    | 192.168.2.99 - - [18/Feb/2018 11:34:23] "GET / HTTP/1.1" 200 -
web_1    | 192.168.2.99 - - [18/Feb/2018 11:34:24] "GET /favicon.ico HTTP/1.1" 404 -
web_1    | 192.168.2.99 - - [18/Feb/2018 11:34:25] "GET / HTTP/1.1" 200 -
web_1    | 192.168.2.99 - - [18/Feb/2018 11:34:26] "GET / HTTP/1.1" 200 -
web_1    | 192.168.2.99 - - [18/Feb/2018 11:34:27] "GET / HTTP/1.1" 200 -
web_1    | 192.168.2.99 - - [18/Feb/2018 11:34:28] "GET / HTTP/1.1" 200 -
web_1    | 192.168.2.99 - - [18/Feb/2018 11:34:28] "GET / HTTP/1.1" 200 -
web_1    | 192.168.2.99 - - [18/Feb/2018 11:34:29] "GET / HTTP/1.1" 200 -
```

**列出相关容器(ps)**
```python
root@consul-app:/tmp/Compose/test1# docker-compose ps
    Name                   Command               State           Ports          
-------------------------------------------------------------------------------
test1_redis_1   docker-entrypoint.sh redis ...   Up      6379/tcp               
test1_web_1     python app.py                    Up      0.0.0.0:5000->5000/tcp 
```

**列出容器内相关进程(top)**
```python
# docker-compose top 
test1_redis_1
  UID      PID    PPID   C   STIME   TTY     TIME         CMD      
------------------------------------------------------------------
libuuid   10031   9998   0   19:52   ?     00:00:00   redis-server 

test1_web_1
UID     PID    PPID    C   STIME   TTY     TIME                 CMD              
--------------------------------------------------------------------------------
root   10027   9994    0   19:52   ?     00:00:00   python app.py                
rhttps://www.gitbook.com/book/lotusching/ops/edit#oot   10124   10027   1   19:52   ?     00:00:02   /usr/local/bin/python app.py 
```

**停止(Stop)**
前台运行使用Ctrl+C停止，后台运行使用`docker-compose down`
```
Gracefully stopping... (press Ctrl+C again to force)
Stopping test1_web_1   ... done
Stopping test1_redis_1 ... done
```


**关闭(Down)**
docker-compose执行down命令时，会清理当前项目建立的相关容器和网络
```
# docker-compose down
Removing test1_web_1   ... done
Removing test1_redis_1 ... done
Removing network test1_default
```