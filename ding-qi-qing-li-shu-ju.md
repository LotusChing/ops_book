# 定期清理数据

## Python版
脚本很简单，定期清理30天前的索引
```python
import re
import os
import logging
import datetime
import traceback
from elasticsearch import Elasticsearch


def init_logger(logger_name, log_file, level=logging.INFO):
    log = logging.getLogger(logger_name)
    formatter = logging.Formatter('%(asctime)s - %(name)s - %(pathname)s:%(lineno)d - %(levelname)s - %(message)s')
    file_handler = logging.FileHandler(log_file)
    stream_handler = logging.StreamHandler()
    stream_handler.setFormatter(formatter)
    file_handler.setLevel(level)
    file_handler.setFormatter(formatter)
    log.setLevel(level)
    log.addHandler(file_handler)
    log.addHandler(stream_handler)


def init_es():
    try:
        current_logger.info('初始化ES连接.')
        es = Elasticsearch(
            ['host'],
            port=9200,
            http_auth=('user', 'password'),
            use_ssl=True,
            verify_certs=False,
            ca_certs='root-ca.pem',
            ssl_assert_hostname=False,
        )
        current_logger.info('连接ES成功.')
        return es
    except Exception as e:
        traceback.print_exc()
        current_logger.error('连接ES失败，错误信息：{}.'.format(str(e)))


def main():
    try:
        current_logger.info('启动清理ES任务.')
        es = init_es()
        ignore_index_list = ['searchguard', '.kibana']
        current_logger.info('获取ES中的索引列表.')
        res = es.cat.indices(format='json')
        expire_30_day_ago = datetime.datetime.now() - datetime.timedelta(days=30)
        expire_index_date = expire_30_day_ago.strftime('%Y.%m.%d')
        pure_index_list = []
        current_logger.info('正则过滤日期信息.')
        for index in res:
            if index['index'] not in ignore_index_list:
                pure_index_name = re.sub(r'-\d{4}.\d{2}.\d{2}', '', index['index'])
                if pure_index_name not in pure_index_list:
                    pure_index_list.append(pure_index_name)
        current_logger.info('过滤日期信息完成.')
        current_logger.info('开始遍历索引列表{}，删除过期索引.'.format(pure_index_list))
        for index in pure_index_list:
            current_logger.info('开始删除索引: [{}-{}]'.format(index, expire_index_date))
            es.indices.delete(index='{}-{}'.format(index, expire_index_date), ignore=[400, 404])
        current_logger.info('清理过期索引完成，即将退出程序.')
    except Exception as e:
        traceback.print_exc()
        current_logger.error('删除索引出现异常，错误信息: {}'.format(str(e)))

if __name__ == '__main__':
    root_path = os.getcwd()
    init_logger('elk', '{}/console.log'.format(root_path))
    current_logger = logging.getLogger('elk')
    main()
```

## Shell版
```shell
#!/bin/bash
es_host="host"
es_port="9200"
es_user="user"
es_pass="passwd"
index_list=`curl -s -k -XGET https://${es_user}:${es_pass}@${es_host}:${es_port}/_cat/indices|egrep -v 'kiban|searchguard'|awk '{ split($3,a,"-20"); print a[1]}'|sort -u`
now=`date +%Y%m%d`
days_30_before=`date -d "$now 30 days ago" +%Y.%m.%d`
for index in ${index_list}
do
  echo "Clean expire index, http://${host}:${port}/${index}-${days_30_before}"
done
```