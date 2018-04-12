# default_type无法传递到logstash
## 背景
我在尝试通过filebeat收集nginx日志和其他系统日志，通过document_type标识类型，然后传送到logstash，logstash根据type判断，最后将日志存储es中以type为关键字的索引中

## 环境
* filebeat-6.2.2
* logstash 6.2.2

## 现象
在logstash配置里无法通过[type]变量到filebeat配置中document_type声明的类型


## 配置		
**filebeat.yml**
```yaml
filebeat:
  prospectors:
      -
        paths:
	  - /opt/tengine/logs/access_blue.log
	document_type: nginx
      -
	paths:
	  - /var/log/syslog
	  - /var/log/auth.log
	document_type: syslog
```

**logstash**
```ruby
input {
  beats {
	port => 5044
  }
}

filter {
  if [type] == "nginx" {
	mutate { add_field => { "index" => "nginx" } }
  }
  else {
	mutate { add_field => { "index" => "other" } }
  }
}

output {
  stdout { codec => rubydebug }
  #elasticsearch {
  #  hosts => ["localhost"]
  #  index => "%{index}-%{host}-%{+YYYY.MM.dd}"
  #}
}
```

## 现象

logstash输出：
```ruby
{
	   "source" => "/opt/tengine/logs/access_blue.log",
	   "message" => "{\"access_path\":\"192.168.1.1\",\"client_ip\":\"192.168.1.1\",\"http_host\":\"test.abc.com\",\"@timestamp\":\"2018-03-07T09:08:58+08:00\",\"method\":\"GET\",\"url\":\"/static/image/favicon.ico\",\"status\":\"200\",\"http_referer\":\"http://test.abc.com/\",\"body_bytes_sent\":\"1150\",\"request_time\":\"0.005\",\"http_user_agent\":\"Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36\",\"total_bytes_sent\":\"1655\",\"server_ip\":\"192.168.1.150\"}",
	  "@version" => "1",
		  "host" => "server",
	"@timestamp" => 2018-03-07T01:48:00.030Z,
		  "beat" => {
		"hostname" => "server",
		 "version" => "6.2.2",
			"name" => "server"
	},
		 "index" => "other",
		"offset" => 496257,
		  "tags" => [
		[0] "beats_input_codec_plain_applied"
	]
}
```


## 解决
通过在filebeat端通过field添加标识信息，见上核心配置

```ruby
root@a9e7c1d83c90:/etc/logstash/conf.d# cat 11-nginx.conf 
filter {
  if [fields][document_type] == "nginx_access" {
    json {
      source => "message"
    }
  }
}
```


```ruby
root@a9e7c1d83c90:/etc/logstash/conf.d# cat 30-output.conf 
output {
  # stdout { codec => rubydebug }
  elasticsearch {
    hosts => ["localhost:9201"]
    index => "%{[fields][document_type]}-%{host}-%{+YYYY.MM.dd}"
  }
}
```