# 快速构建ELK环境

## 基础环境


## compose文件

**docker版本17以上**
```docker
version: '3.2'
services:
  elk:
    container_name: elk
    image: sebp/elk:latest
    ports:
      - "local_ip_addr:5601:5601" 
      - "local_ip_addr:9200:9200"
      - "local_ip_addr:5044:5044"
    environment:
      - TZ=Asia/Shanghai
      - CLUSTER_NAME=wecan
    volumes:
      - type: bind
        source: /opt/elk/logstash/conf.d
        target: /etc/logstash/conf.d
      - type: bind
        source: /opt/elk/elasticsearch/data
        target: /var/lib/elasticsearch
```

**docker版本17以下**
```docker
version: '2'
services:
  elk:
    container_name: elk
    image: sebp/elk:latest
    ports:
      - "192.168.1.150:5601:5601" 
      - "192.168.1.150:9200:9200"
      - "192.168.1.150:5044:5044"
    environment:
      - TZ=Asia/Shanghai
      - CLUSTER_NAME=wecan
    volumes:
      -  /opt/elk/logstash/conf.d:/etc/logstash/conf.d
      -  /opt/elk/elasticsearch/data:/var/lib/elasticsearch
```

