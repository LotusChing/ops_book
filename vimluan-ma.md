# vim乱码
首先确保中文字体以及系统环境locale是utf8编码
```shell
cat ~/.vimrc 
set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936
set termencoding=utf-8
set encoding=utf-8
```