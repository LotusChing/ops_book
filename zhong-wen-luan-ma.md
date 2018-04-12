# 中文乱码

1. 拷贝windows行的字体，上传到zabbix前端服务器 PS: C:\Windows\Fonts
2. 修改前端文件使用中文字体
```shell
vi /usr/share/zabbix/include/defines.inc.php
:%s/graphfont/simhei/g
```
3. 刷新页面