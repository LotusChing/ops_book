# General Log

## 用处
* 调试
* 分析业务代码逻辑
* 审计


## 相关参数
```
mysql> set general_log=1;
```


## 问题
1. 千万不要随随便便的在线上开启这个参数，不然你会倒霉的~
2. 如果开启了，就一定要记得关闭它
3. mysql5.7社区版暂时不支持审计功能，不过可以通过MariaDB的`server_audit_plugin`实现类似的功能

