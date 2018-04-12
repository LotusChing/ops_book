# 快速安装

参考：https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html

**deb**
```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.2.2-amd64.deb
sudo dpkg -i filebeat-6.2.2-amd64.deb
```

**rpm**
```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.2.2-x86_64.rpm
sudo rpm -vi filebeat-6.2.2-x86_64.rpm
```

**mac**
```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.2.2-darwin-x86_64.tar.gz
tar xzvf filebeat-6.2.2-darwin-x86_64.tar.gz
```

**docker**
```docker
docker pull docker.elastic.co/beats/filebeat:latest
```
