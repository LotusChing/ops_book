# Search Guard

## Search Guard 是什么？

## Search Guard 能做什么？

## Search Guard 部署

### 1.ES安装Search Guard插件
版本选择：https://oss.sonatype.org/content/repositories/releases/com/floragunn/search-guard-6/
```
wget https://oss.sonatype.org/content/repositories/releases/com/floragunn/search-guard-6/6.2.2-21.0/search-guard-6-6.2.2-21.0.zip
/opt/elasticsearch/bin/elasticsearch-plugin install file:/var/lib/elasticsearch/search-guard-6-6.2.2-21.0.zip
```

### 2. 快速生成CA以及客户端证书
拉取工具库
```
git clone -b es-6.2.2 https://github.com/floragunncom/search-guard-ssl.git
cd /var/lib/elasticsearch/search-guard-ssl/example-pki-scripts/
```

修改脚本
```
cat /opt/elk/elasticsearch/data/search-guard-ssl/example-pki-scripts/example.sh 
#!/bin/bash
OPENSSL_VER="$(openssl version)"

if [[ $OPENSSL_VER == *"0.9"* ]]; then
	echo "Your OpenSSL version is too old: $OPENSSL_VER"
	echo "Please install version 1.0.1 or later"
	exit -1
else
    echo "Your OpenSSL version is: $OPENSSL_VER"
fi
set -e
tag="test"
./clean.sh
./gen_root_ca.sh ${tag}_elk_ca_pwd ${tag}_elk_ts_pwd
./gen_node_cert.sh 1 ${tag}_elk_ks_pwd ${tag}_elk_ca_pwd
./gen_client_node_cert.sh ${tag} ${tag}_elk_c_node_pwd ${tag}_elk_ca_pwd
```

执行脚本创建CA私钥、证书等
```
bash example.sh
cp node-1-keystore.jks  truststore.jks /etc/elasticsearch/
cp tag-keystore.jks /etc/elasticsearch/
cp tag-keystore.jks /opt/elasticsearch/plugins/search-guard-6/sgconfig/
```

### 3. 配置ES启用Search Guard
```yml
root@a9e7c1d83c90:/# egrep -v "^$|#" /etc/elasticsearch/elasticsearch.yml 
cluster.name: test
node.name: node-1
network.host: 0.0.0.0
network.publish_host: 192.168.1.150
http.port: 9200
searchguard.ssl.transport.keystore_filepath: node-1-keystore.jks
searchguard.ssl.transport.keystore_password: tag_elk_ks_pwd
searchguard.ssl.transport.truststore_filepath: truststore.jks
searchguard.ssl.transport.truststore_password: tag_elk_ts_pwd
searchguard.ssl.transport.enforce_hostname_verification: false
searchguard.authcz.admin_dn:
  - "CN=tag, OU=client, O=client, L=Test, C=DE"
searchguard.ssl.http.enabled: true
searchguard.ssl.http.keystore_filepath: node-1-keystore.jks
searchguard.ssl.http.keystore_password: tag_elk_ks_pwd
searchguard.ssl.http.truststore_filepath: truststore.jks
searchguard.ssl.http.truststore_password: tag_elk_ts_pwd
searchguard.enterprise_modules_enabled: false
```

### 4. 同步当前认证信息到ES中
```
cd /opt/elasticsearch/plugins/search-guard-6/
chmod +x tools/*.sh
tools/sgadmin.sh -ts /etc/elasticsearch/truststore.jks -tspass tag_elk_ts_pwd -ks sgconfig/tag-keystore.jks -kspass tag_elk_c_node_pwd -cd sgconfig/ -icl -nhnv -h localhost
```

### 5. 修改Logstash中输出配置
修改配置，安装常用插件
```ruby
cat /etc/logstash/conf.d/30-output.conf 
output {
  # stdout { codec => rubydebug }
  elasticsearch {
    user => user
    password => passwd
    ssl => true
    ssl_certificate_verification => true
    truststore => "/etc/elasticsearch/truststore.jks"
    truststore_password => tag_elk_ts_pwd
    hosts => ["localhost:9200"]
    index => "%{[fields][document_index]}-%{host}-%{+YYYY.MM.dd}"
  }
}

/opt/logstash/bin/logstash-plugin install logstash-filter-multiline
```

### 6. Kibana安装Search Guard插件
版本选择：https://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.floragunn%22%20AND%20a%3A%22search-guard-kibana-plugin%22
```
wget https://search.maven.org/remotecontent?filepath=com/floragunn/search-guard-kibana-plugin/6.2.2-10/search-guard-kibana-plugin-6.2.2-10.zip
/opt/kibana/bin/kibana-plugin install file:/var/lib/elasticsearch/search-guard-kibana-plugin-6.2.2-10.zip
```

### 7. 修改Kibana中ES数据源配置
```
egrep -v "^$|#" /opt/kibana/config/kibana.yml 
server.host: "0.0.0.0"
elasticsearch.ssl.verificationMode: "certificate"
elasticsearch.username: "user"
elasticsearch.password: "passwd"
elasticsearch.url: "https://localhost:9200"
elasticsearch.ssl.certificateAuthorities: /var/lib/elasticsearch/search-guard-ssl/example-pki-scripts/ca/root-ca.pem
elasticsearch.requestHeadersWhitelist: ["sgtenant","Authorization"]
searchguard.multitenancy.tenants.preferred: ["admin","kibanaserver", "logstash", "kibanaro"]
searchguard.multitenancy.enabled: true
```

### 8. 重启各服务检验配置有效

### 9. 遇到的问题
  **1. Kibana重复检验登录**
  如果Kibana出现重复校验登录，更换浏览器即可

  **2. Kibana安装插件卡死**
  建议用配置较高的主机安装，不然会一直卡在生成浏览器缓存那块
  
  