# 实战案例：解析处理Nginx日志到ES

## 准备工作

区分两种情况
1. nginx日志为json格式
2. nginx日志不是json，则需要写pattern匹配

### 1. 设置nginx日志为json格式
为了简单，所以直接格式化成json
```nginx
```

### 2. 如果需要获取geo_ip的话，补充配置
```nginx
```

### 3. 下载最新的GeoIP2 database
```bash
```

### 4. 配置logstash解析nginx日志
```ruby
```

### 5. 检查es数据
```bash
```

### 6.Kibana配置常用查询
* 特定虚拟主机
* 异常状态请求

### 7. Kibana配置常用Visualize
* 状态码
* 地区

