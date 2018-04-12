# 攻击手段

## 撞库
1. 什么是**撞库？**


2. **如何防范？**
* 提高密码复杂度、避免单一性
* 固定人员IP，使用access_module进行访问控制



## 文件上传
因为结尾是1.php，nginx会将1.jpg当作php代码执行
```
http://www.test.com/upload/1.jpg/1.php
```

避免上述问题
```
location ^~ /upload {
    root /opt/app/images/;
    if ($request_filename ~* (.*)\.php) {
        return 403;
    }
}
```

## SQL注入

环境演示

**1. 准备库/表/数据**

    ```
    mysql> show create table info.user\G
    *************************** 1. row ***************************
           Table: user
    Create Table: CREATE TABLE `user` (
      `id` int(11) DEFAULT NULL,
      `username` varchar(64) DEFAULT NULL,
      `password` varchar(64) DEFAULT NULL,
      `email` varchar(64) DEFAULT NULL
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8
    1 row in set (0.01 sec)
    mysql> select * from info.user;
    +------+----------+----------+------------+
    | id   | username | password | email      |
    +------+----------+----------+------------+
    |    1 | Da       | 123123   | da@da.como |
    |    2 | Yo       | 123123   | yo@yo.com  |
    |    3 | lc       | 123123   | lc@lc.com  |
    +------+----------+----------+------------+
    
    ```


**2. flask模拟后台服务, `python35 app.py`**

    ```
    import pymysql
    from flask import Flask, request
    app = Flask(__name__)
    
    @app.route('/')
    def index():
        return "Flask."
    
    @app.route('/login', methods=['GET', 'POST'])
    def login():
        username = request.form.get('username')
        password = request.form.get('password')
        conn = pymysql.connect(host="localhost", user="root", password="", db="info", charset="utf8mb4", cursorclass=pymysql.cursors.DictCursor)
        cursor = conn.cursor()
        sql = "select id from user where username='{}' and password='{}'".format(username, password)
        cursor.execute(sql)
        result = cursor.fetchone()
        return ("SQL: {}<br /><br />Result:{}".format(sql, "Success!" if result else "Failed!"))
    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=5001, debug=True)
    ```

**3. ngixn虚拟主机配置，配置完reload**

  ```
      server {
        listen     80;
        server_name www.sql_injection.com;
        root   /opt/app/code;
    
        location ~\.(html|htm) {
            root  /opt/app/sql_injection/;
            index  index.html;
        } 
        location / {
            proxy_pass http://127.0.0.1:5001;
        }
    
    }
  ```


**4. html页面文件**
    
    ```
    [root@bj-vmware-test1 conf.d]# cat /opt/app/sql_injection/index.html 
    <form action="/login" method="POST">
        UserName: <input name="username" type="text"/>
        Password: <input name="password" type="text"/>
        <button type="commit" >commit</button>
    </form>
    ```

**5. 测试登录成功/失败**
**成功**
    ```
    SQL: select id from user where username='da' and password='123123'
    
    Result:Success!
    ```
    
    
**失败**
    ```
    SQL: select id from user where username='da' and password='123123a'
    
    Result:Failed!
    ```

**6. 测试注入**

```
SQL: select id from user where username='da' and password='' or 1=1#'

Result:Success!
```



**7. 使用nginx+lua过滤sql注入**

    配置使用waf防护墙
    ```
    git clone https://github.com/loveshell/ngx_lua_waf.git
    cp -rf ngx_lua_waf nginx主目录/waf
    vim /etc/nginx/waf/config.lua # 修改配置
    ```
    
    添加规则
    ```
    [root@bj-vmware-test1 sql_injection]# cat /etc/nginx/waf/wafconf/post 
    \sor\s+
    select.+(from|limit)
    ...
    ```
    
    配置nginx使用waf, 配置完reload
    
    ```
    [root@bj-vmware-test1 sql_injection]# cat /etc/nginx/nginx.conf
    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
    
        lua_package_path "/etc/nginx/waf/?.lua";
        lua_shared_dict limit 10m;
        init_by_lua_file /etc/nginx/waf/init.lua;
        access_by_lua_file /etc/nginx/waf/waf.lua;
        ...
    }
    ```


**8. 再次测试**
![](/assets/post_sql_injection_waf.jpg)

## CC
