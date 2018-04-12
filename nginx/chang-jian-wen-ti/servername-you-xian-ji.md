## conflicting server_name 优先级

当nginx配置文件中有多个相同的server_name时，nginx会**优先匹配先读取到的server**

**什么叫先读取到的server？**
* 如果server被单独的配置文件中，读取顺序是按照包含相同的server_name的配置文件的文件名排序
* 如果冲突的server配置在一个配置文件中，读取顺序就是文件内容的顺序
