# 核心配置

## filebeat 做了什么？
Filebeat 主要做了两件事情

1. 从文件系统获取日志
2. 推送日志到指定位置(logstash、es)

所以主要配置也是围绕着这两个事情的


## 配置示例
```docker
➜  ~ egrep -v "#|^$" /etc/filebeat/filebeat.yml
filebeat:
  prospectors:
    -
      paths:
        - /opt/tengine/logs/*.log
      fields:
        document_type: nginx_access
    - 
      paths:
        - /prodata/www/wbs/logs/WBS*/*.log
      multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
      multiline.negate: true
      multiline.match: after
      multiline.timeout: 10s
      fields:
        document_type: dev_wbs_project
    -
      paths:
        - /prodata/www/uair/logs/localhost_access_log.*.txt
      fields:
        document_type: tomcat_access
        
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
setup.template.settings:
  index.number_of_shards: 3
setup.kibana:
output.logstash:
  hosts: ["127.0.0.1:5044"]
logging.level: debug
```

可以从配置看到几个参数配置
* filebeat.prospectors
  * path：就是定义监控哪些路径下文件
  * fields：添加字段标识这个日志，后面logstash会用到
  * multiline.pattern：多行处理的格式
  * multiline.negate：见图
  * ultiline.match: 见图
  
  ![](/assets/filebeat_multiline_negate_match.jpg)
* output.logstash: logstash地址
* logging.level: 日志级别，debug方便调试
