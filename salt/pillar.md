# Pillar


## Pillar是什么？
Pillar和Grains一样，都是Salt的数据系统，提供数据信息的

## Pillar和Grains的区别是？
* Grains是静态的(宽松意义上)
* Pillar是动态的


* Grains只能定义简单数据`k:v`
* Pillar可以定义复杂数据`k:{'K':v}`


## Pillar的应用场景？
1. 用于给特定的服务器定义特定的数据，并且此类数据属于偏敏感的数据
2. 目标选择



## Pillar如何定义数据？
1. 配置pillar和配置状态文件很像，首先启用pillar

 ```
# vim /etc/salt/master
 779 pillar_roots:
 780   base:
 781     - /srv/pillar
 ```

2. 接着新建目录

 ```
 # mkdir -p /srv/pillar/initial
 ```

3. 然后写pillar文件，文件格式很简单，就是k: v

 ```
 # /srv/pillar# cat initial/basic.sls 
 web_server: nginx
 db_server: mysql
 ```

4. 同时pillar文件支持jinjia2语法，例如根据不同系统设置不同数据

 ```
 root@server:/srv/pillar/initial# cat basic.sls
 {% if grains['os'] == 'Ubuntu' %}
 web_server: nginx
 db_server: mysql
 {% elif grins['os'] == 'CentOS' %}
 web_server: apache
 {% endif %}
 ```

5. 写topfile，必须写，不然将数据定义到指定的minion上
 ```
 root@server:/srv/pillar# cat top.sls 
 base:
   'server':
     - initial.basic
 ```


6. 定义完topfile之后一样要进行刷新
 ```
 root@server:/srv/pillar/initial# salt '*' saltutil.refresh_pillar
 server:
     True
 ```

7. 尝试获取获取数据
 ```
 root@server:/srv/pillar/initial# salt '*' pillar.item web_server
 server:
     ----------
     web_server:
         nginx
 ```




## 如何定义多层级的数据？
 利用缩进来完成多层级数据的定义
 ```
 root@server:/srv/pillar# cat initial/basic.sls
 service:
   {% if grains['os'] == 'Ubuntu' %}
   web_server: nginx
   db_server: mysql
   {% elif grins['os'] == 'CentOS' %}
   web_server: apache
   {% endif %}
 ```

查看数据
 ```
 # salt '*' pillar.item service
 server:
     ----------
     service:
         ----------
         db_server:
             mysql
         web_server:
             nginx
 ```

获取指定层级的数据
 ```
 root@server:/srv/pillar# salt '*' pillar.get service:db_server
 server:
     mysql
 ```


## Pillar怎么作为目标选择的条件？
**无语，没找到怎么将多层级的数据作为条件，这里的例子只要用简单的了**

修改pillar文件
 ```
 root@server:/srv/pillar# cat initial/basic.sls
 cat initial/basic.sls
 {% if grains['os'] == 'Ubuntu' %}
 web_server: nginx
 db_server: mysql
 {% elif grins['os'] == 'CentOS' %}
 web_server: apache
 {% endif %}
 ```

刷新pillar配置，然后在web_server是nginx的minion执行命令
 ```
 root@server:/srv/pillar# salt '*' saltutil.refresh_pillar
 server:
     True
 root@server:/srv/pillar# salt -I 'web_server:nginx' cmd.run 'w'
 server:
      16:41:36 up 51 days,  6:24,  2 users,  load average: 0.31,  0.27, 0.25
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
     root     pts/20   192.168.1.1      09:56    2:40m  0.13s  0.06s bash
     root     pts/4    192.168.1.1      09:15    8.00s  1.33s  0.08s -zsh
 ```

------
隔日更新
找到目标选择中pillar多级数据作为条件的方法了

pillar状态文件和topfile
 ```
 root@server:~# cat /srv/pillar/top.sls 
 base:
   'server':
     - initial.basic
 root@server:~# cat /srv/pillar/initial/basic.sls
 service:
   {% if grains['os'] == 'Ubuntu' %}
     web_server: nginx
     db_server: mysql
   {% elif grins['os'] == 'CentOS' %}
    web_server: apache
  {% endif %}
 ```

命令很简单就是每级用:隔开就行了
 ```
 root@server:~# salt -I 'service:web_server:nginx' cmd.run 'free -m'
 server:
                  total       used       free     shared    buffers     cached
     Mem:         16004      15417        586         13        203       6122
     -/+ buffers/cache:       9091       6912
    Swap:        16336       2922      13414
 ```



## Grains和Pillar汇总对比
|数据系统|类型|采集方式|使用场景|定位位置|
|:-----:|:--:|:-----:|:-----:|
|Grains|静态(也可实现动态)|启动时采集(手动推送)|数据查询 目标选择 配置管理|minion/master|
|Pillar|动态|master定义|目标选择 配置管理 敏感数据存储|master|




