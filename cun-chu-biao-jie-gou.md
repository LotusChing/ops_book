# 存储表结构

* items: 存储监控项的基础信息
* history: 存储类型numeric(float)监控数据
* history_str: 存储类型character(up to 255 bytes)监控数据
* history_uint:存储类型numeric(unsigned integers)监控数据
* history_text:存储类型text监控数据
* history_log: 存储类型log监控数据
* trends: 存储类型为numeric(float)的趋势数据，基于history得出max、min、avg等汇总趋势数据
* trends_uint: 存储类型为numeric(unsigned integers)的趋势数据，基于histroy_uint中得出max、min、avg等汇总趋势数据

