# Summary

* [Introduction](README.md)
* Consul
  * [consul + consul-template + nginx](consul-+-consul-template-+-nginx.md)
* Linux
  * [basic](basic.md)
  * [service](service.md)
    * [Tomcat](service/tomcat.md)
      * [GC](service/tomcat/gc.md)
  * [detail](detail.md)
    * [文件读写](detail/fileio.md)
  * [Tips](tips.md)
  * [常见问题](chang-jian-wen-ti.md)
    * [vim乱码](vimluan-ma.md)
* DB
  * [mysql](mysql.md)
    * [install](mysql/install.md)
      * [5.7](mysql/57.md)
    * [upgrade](mysql/upgrade.md)
      * [5.6 -&gt; 5.7](mysql/upgrade/56-57.md)
    * [小技巧](mysql/xiao-ji-qiao.md)
      * [免密](mysql/xiao-ji-qiao/mian-mi.md)
      * [多实例](mysql/xiao-ji-qiao/duo-shi-li.md)
    * [grant](mysql/grant.md)
      * [role](mysql/grant/role.md)
    * [files](mysql/files.md)
      * [logs](mysql/files/logs.md)
        * [slow log](mysql/files/slow-log.md)
        * [general log](mysql/files/general-log.md)
          * [Audit in MySQL 5.7](mysql/files/general-log/mysql57shi-yong-mariadb-audit-plugin-cha-jian.md)
    * [storage engine](mysql/storage-engine.md)
      * [csv](mysql/storage-engine/csv.md)
      * [myisam](mysql/storage-engine/myisam.md)
      * [innodb](mysql/storage-engine/innodb.md)
      * [federated](mysql/storage-engine/federated.md)
    * [data type](data-type.md)
      * [整数](mysql/zheng-shu.md)
      * [浮点数](mysql/fu-dian-shu.md)
      * [字符串](mysql/zi-fu-chuan.md)
      * [枚举、集合](mysql/mei-ju-3001-ji-he.md)
      * [日期](mysql/ri-qi.md)
      * [json](mysql/json.md)
    * [临时表](mysql/biao-guan-li.md)
    * [外键](mysql/wai-jian.md)
    * [备份](mysql/bei-fen.md)
  * [pgsql](pgsql.md)
  * [redis](redis.md)
    * [安装](redis/an-zhuang.md)
    * [配置、启动、关闭](redis/pei-zhi-3001-qi-dong-3001-guan-bi.md)
    * [键操作](redis/jian-cao-zuo.md)
    * [数据结构](redis/shu-ju-jie-gou.md)
      * [string\(字符串\)](redis/shu-ju-jie-gou/stringzi-fu-4e3229.md)
      * [hash\(哈希\)](redis/shu-ju-jie-gou/hashha-5e0c29.md)
      * [list\(列表\)](redis/shu-ju-jie-gou/listlie-886829.md)
      * [set\(集合\)](redis/shu-ju-jie-gou/setji-540829.md)
      * [zset\(有序集合\)](redis/shu-ju-jie-gou/zsetyou-xu-ji-540829.md)
    * [慢查询](redis/man-cha-xun.md)
    * [Redis Shell](redis/redis-shell.md)
    * [Pipeline](redis/pipeline.md)
    * [事务和Lua](redis/shi-wu-he-lua.md)
    * Bitmaps
    * [HyperLogLog](redis/hyperloglog.md)
    * [客户端](redis/ke-hu-duan.md)
      * [客户端通信协议](redis/ke-hu-duan/ke-hu-duan-tong-xin-xie-yi.md)
      * [Python客户端redis-py](redis/ke-hu-duan/pythonke-hu-duan-redis-py.md)
      * [客户端管理](redis/ke-hu-duan/ke-hu-duan-guan-li.md)
    * [持久化](redis/chi-jiu-hua.md)
      * [RDB](redis/chi-jiu-hua/rdb.md)
      * [AOF](redis/chi-jiu-hua/aof.md)
      * [RDB、AOF比较](redis/chi-jiu-hua/rdbaofbi-jiao.md)
      * [问题定位、优化](redis/chi-jiu-hua/wen-ti-ding-wei-3001-you-hua.md)
    * [复制](redis/fu-zhi.md)
      * [复制配置](redis/fu-zhi-pei-zhi.md)
      * [复制拓扑](redis/fu-zhi-tuo-pu.md)
      * [复制原理](redis/fu-zhi-yuan-li.md)
      * [复制问题](redis/fu-zhi-wen-ti.md)
    * [Redis Sentinel](redis/redis-sentinel.md)
      * [环境配置](redis/redis-sentinel/huan-jing-pei-zhi.md)
      * [哨兵配置](redis/redis-sentinel/shao-bing-pei-zhi.md)
      * [模拟故障](redis/redis-sentinel/mo-ni-gu-zhang.md)
      * 实现原理
      * 常见问题
      * [遗留问题](redis/redis-sentinel/yi-liu-wen-ti.md)
    * [面试题](redis/mian-shi-ti.md)
  * mongo
  * influxdb
* [Monitor](monitor.md)
  * [os](os.md)
  * [network](network.md)
  * [service](service.md)
  * [api](api.md)
  * [log](log.md)
  * distributed
  * [reference](reference.md)
    * [创业型公司如何做好监控报警](reference/创业型公司如何做好监控报警.md)
  * [思考](si-kao.md)
* Arch
  * lb
    * SLB
    * LVS
    * NGINX
    * HAProxy
  * [ha](ha.md)
    * [MySQL高可用选型](ha/mysqlgao-ke-yong-xuan-xing.md)
  * 接入层
    * [Nginx](nginx.md)
      * [安装](nginx/an-zhuang.md)
        * [Tengine](nginx/an-zhuang/tengine.md)
      * [常见变量](nginx/chang-jian-bian-liang.md)
      * 目录文件
      * 配置文件
      * [日志格式](nginx/ri-zhi-ge-shi.md)
      * 模块
        * [stub\_status](nginx/stubstatus.md)
        * [random\_index\_module](nginx/randomindex-module.md)
        * [sub\_filter](nginx/subfilter.md)
        * [limit\_conn & limit\_req](nginx/limitconnand-limit-req.md)
        * [auth\_basic ](nginx/authbasic.md)
        * [access\_module](nginx/accessmodule.md)
        * [secure\_link](nginx/securelink.md)
        * [geoip](nginx/geoip.md)
        * [dyups](nginx/dyups.md)
      * [请求限制](nginx/qing-qiu-xian-zhi.md)
      * 访问控制
      * [应用场景](nginx/ying-yong-chang-jing.md)
        * [静态资源服务](nginx/jing-tai-zi-yuan-fu-wu.md)
        * [代理服务](nginx/dai-li-fu-wu.md)
        * [负载均衡服务](nginx/fu-zai-jun-heng-fu-wu.md)
          * [后端容错配置](nginx/fu-zai-jun-heng-fu-wu/hou-duan-rong-cuo-pei-zhi.md)
        * [缓存服务](nginx/huan-cun-fu-wu.md)
      * [动静分离](nginx/dong-jing-fen-li.md)
      * [rewrite](nginx/rewrite.md)
      * HTTPS
        * 单向证书验证
        * [双向证书验证](nginx/shuang-xiang-zheng-shu-yan-zheng.md)
      * [nginx lua](nginx/nginx-lua.md)
        * [lua 基础语法](nginx/nginx-lua/lua-ji-chu-yu-fa.md)
        * [nginx + lua 环境安装](nginx/nginx-lua/nginx-+-lua-huan-jing-an-zhuang.md)
        * [nginx调用lua指令](nginx/nginx-lua/nginxdiao-yong-lua-zhi-ling.md)
        * [实现灰度发布](nginx/nginx-lua/shi-xian-hui-du-fa-bu.md)
        * 实现流量复制
      * [指令执行顺序](nginx/zhi-ling-zhi-xing-shun-xu.md)
      * [常见问题](nginx/chang-jian-wen-ti.md)
        * [server\_name 优先级](nginx/chang-jian-wen-ti/servername-you-xian-ji.md)
        * [location 优先级](nginx/chang-jian-wen-ti/location-you-xian-ji.md)
        * [alias和root区别](nginx/chang-jian-wen-ti/aliashe-root-qu-bie.md)
        * [获取客户端精确IP](nginx/chang-jian-wen-ti/huo-qu-ke-hu-duan-jing-que-ip.md)
        * [常见错误码](nginx/chang-jian-wen-ti/chang-jian-cuo-wu-ma.md)
      * [try\_files](nginx/tryfiles.md)
      * [性能优化](nginx/xing-neng-you-hua.md)
      * [Nginx安全](nginx/nginxan-quan.md)
        * [恶意行为](nginx/nginxan-quan/e-yi-xing-wei.md)
        * [攻击手段](nginx/nginxan-quan/gong-ji-shou-duan.md)
* Docker
  * 基础
    * [镜像导入导出](jing-xiang-dao-ru-dao-chu.md)
  * [进阶](jin-jie.md)
    * [Compose](jin-jie/compose.md)
      * [构建Flask+Redis](jin-jie/compose/gou-jian-flask-+-redis.md)
      * [构建LNMP](jin-jie/compose/gou-jian-lnmp.md)
      * [构建Nginx代理Tomcat集群](jin-jie/compose/gou-jian-nginx-dai-li-tomcat-ji-qun.md)
      * [快速构建ELK](jin-jie/compose/kuai-su-gou-jian-elk.md)
      * 常见问题
        * [Nginx无法获取真实客户端IP](jin-jie/compose/nginxwu-fa-huo-qu-zhen-shi-ke-hu-duan-ip.md)
      * [小技巧](jin-jie/compose/xiao-ji-qiao.md)
    * 网络
      * [端口映射](jin-jie/duan-kou-ying-she.md)
      * [容器网络](jin-jie/rong-qi-wang-luo.md)
      * [Macvlan方案实现跨主机容器通信](jin-jie/macvlanfang-an-shi-xian-kua-zhu-ji-rong-qi-tong-xin.md)
      * [Weave方案实现跨主机容器通信](jin-jie/weavefang-an-shi-xian-kua-zhu-ji-rong-qi-tong-xin.md)
  * K8S
  * Swarm
    * [Swarm 开始](swarm-kai-shi.md)
    * [节点管理](jie-dian-guan-li.md)
    * [服务管理](fu-wu-guan-li.md)
    * [Overlay 网络](overlay-wang-luo.md)
    * [数据持久化](shu-ju-chi-jiu-hua.md)
    * [负载均衡](fu-zai-jun-heng.md)
    * [配置文件存储](pei-zhi-wen-jian-cun-chu.md)
    * [管理节点高可用](guan-li-jie-dian-gao-ke-yong.md)
    * 实战案例
      * [可扩展的运维平台](ke-kuo-zhan-de-yun-wei-ping-tai.md)
    * [常见问题](chang-jian-wen-ti.md)
  * 其他
    * [合理的进行Docker问题的求助](he-li-de-jin-xing-docker-wen-ti-de-qiu-zhu.md)
* Deployment
  * shell
  * [jenkins](jenkins.md)
    * pipeline
    * 常用插件
    * 常见问题
      * [记saltstack插件的一个坑](jenkins/ji-saltstack-cha-jian-de-yi-ge-keng.md)
  * webhooks
  * [Salt](salt.md)
    * [install](salt/install.md)
    * [状态文件](salt/zhuang-tai-wen-jian.md)
    * [zmq](salt/zmq.md)
    * [grains](salt/grains.md)
    * [pillar](salt/pillar.md)
    * [远程执行](salt/yuan-cheng-zhi-xing.md)
      * [目标选择](salt/yuan-cheng-zhi-xing/mu-biao-xuan-ze.md)
      * [执行模块](salt/yuan-cheng-zhi-xing/zhi-xing-mo-kuai.md)
      * [返回程序](salt/yuan-cheng-zhi-xing/fan-hui-cheng-xu.md)
      * [手撸模块](salt/yuan-cheng-zhi-xing/shou-lu-mo-kuai.md)
    * [配置管理](salt/pei-zhi-guan-li.md)
    * salt-api
* [CMDB](cmdb.md)
  * play with monitor
  * play with salt
* [Network](network.md)
  * Protocol
    * [HTTP](http.md)
    * [HTTPS](https.md)
      * [get certificate](https/get-certificate.md)
      * reference
        * [HTTPS科普扫盲帖](https/https.md)
      * [双向认证](https/shuang-xiang-ren-zheng.md)
  * [Basic](basic.md)
    * [以太网](basic/yi-tai-wang.md)
    * [IP 协议](basic/ip-xie-yi.md)
    * [ICMP](basic/icmp.md)
    * [UDP](basic/udp.md)
    * [TCP](basic/tcp.md)
      * [流的概念](basic/tcp/liu-de-gai-nian.md)
      * [可靠性](basic/tcp/ke-kao-xing.md)
      * [滑动窗口](basic/tcp/hua-dong-chuang-kou.md)
      * [连接建立](basic/tcp/lian-jie-jian-li.md)
      * [滑窗细节](basic/tcp/hua-chuang-xi-jie.md)
      * [重新发送](basic/tcp/zhong-xin-fa-song.md)
* [Automatic Ops Post](automatic-ops-post.md)
  * [如烹小虾： 运维自动化闭环，腾讯是这样做的](ru-peng-xiao-xia-ff1a-yun-wei-zi-dong-hua-bi-huan-ff0c-teng-xun-shi-zhe-yang-zuo-de.md)
* [Capacity Planning](capacity-planning.md)
  * [Linux Server](capacity-planning/linux-server.md)
    * [CPU](capacity-planning/cpu.md)
    * [File IO](capacity-planning/file-io.md)
    * [Network](capacity-planning/network.md)
    * [OS](capacity-planning/os.md)
  * [ECS Test](capacity-planning/ecs-test.md)
    * [Computing power](capacity-planning/ecs-test/computing-power.md)
      * [系列1 2核4G](capacity-planning/ecs-test/xi-lie-1-2-he-4g.md)
      * [系列2 2核4G\(共享\)](capacity-planning/ecs-test/xi-lie-2-2-he-4g.md)
      * [系列2 2核4G\(独享\)](capacity-planning/ecs-test/xi-lie-2-2-he-4g-du-4eab29.md)
      * [系列3 2核4G\(共享计算\)](capacity-planning/ecs-test/xi-lie-3-2-he-4g-gong-xiang-ji-7b9729.md)
      * [系列3 2核8G\(共享通用\)](capacity-planning/ecs-test/xi-lie-3-2-he-8g-gong-xiang-tong-752829.md)
    * File IO
      * [高效云盘](capacity-planning/ecs-test/gao-xiao-yun-pan.md)
      * [SSD](capacity-planning/ecs-test/ssd.md)
      * [普通云盘](capacity-planning/ecs-test/pu-tong-yun-pan.md)
    * [Network](capacity-planning/ecs-test/network.md)
      * [Direct](capacity-planning/ecs-test/direct.md)
      * [SLB -&gt; ECS](capacity-planning/ecs-test/slb-ecs.md)
  * [性能测试](capacity-planning/xing-neng-ce-shi.md)
  * [基础概念](capacity-planning/ji-chu-gai-nian.md)
  * 测试
    * [测试分类](capacity-planning/ce-shi-fen-lei.md)
* Security
  * [Injection](injection.md)
    * [SQL Injection](injection/sql-injection.md)
  * [XSS](xss.md)
  * [CSRF](csrf.md)
  * [Security By Default](security-by-default.md)
  * [OWASP Dependency Check](owasp-dependency-check.md)
    * Java
    * Python
    * 前端
  * [WAF](waf.md)
  * [DDos](ddos.md)
* [Search](search.md)
  * ELK
    * logstash
    * elastic search
    * kibana
* Tomcat
  * [环境安装](huan-jing-an-zhuang.md)
  * [配置说明](pei-zhi-shuo-ming.md)
  * 常见问题
    * [X-Real-IP](x-real-ip.md)
    * [假死问题汇总](jia-si-wen-ti-hui-zong.md)
    * [access log json格式](access-log-jsonge-shi.md)
  * [日志配置](ri-zhi-pei-zhi.md)
* ELK
  * 快速开始
  * logstash
    * 快速安装
    * 工作流程
    * 常用插件
    * [实战案例：解析处理Nginx日志到ES](shi-zhan-an-li-ff1a-jie-xi-chu-li-nginx-ri-zhi-dao-es.md)
    * 实战案例：解析处理Tomcat日志到ES
    * 实战案例：解析处理Python日志到ES
    * 安全配置
    * 常见问题
  * [kibana](kibana.md)
    * 快速安装
    * 核心配置
    * [查询语法](kibana/cha-xun-yu-fa.md)
    * Visualize
    * Dashboard
    * 常见问题
    * 安全配置
  * filebeat
    * [快速安装](kuai-su-an-zhuang.md)
    * [核心配置](he-xin-pei-zhi.md)
    * 常见问题
      * [default\_type无法传递到logstash](defaulttype-wu-fa-chuan-di-dao-logstash.md)
  * [学习资料](xue-xi-zi-liao.md)
  * elasticsearch
    * 快速安装
    * 目录结构
    * 核心配置
    * 术语名词
    * 搜索语句
      * curl操作es
    * 结构化查询语句DSL
    * 集群配置
    * 安全配置
      * [Search Guard](search-guard.md)
    * 常见问题
      * [定期清理数据](ding-qi-qing-li-shu-ju.md)
    * 映射、模版
* [其他](qi-ta.md)
  * [快速进入状态](kuai-su-jin-ru-zhuang-tai.md)
* MQ
  * [RabbitMQ](rabbitmq.md)
* 故障排除
  * [启动故障](qi-dong-gu-zhang.md)
  * 文件系统故障
  * 网络故障
  * DNS故障
  * Web故障
  * MySQL故障
  * 硬件故障
* 面试
  * Linux相关
    * buffers & cached
  * 网络相关
  * Nginx相关
    * 参数解释
    * 工作原理
  * Tomcat相关
  * JVM相关
  * MySQL相关
    * 主从原理
  * Redis相关
  * MongoDB相关
  * Shell相关
  * 架构相关
    * 高可用架构
    * 负载均衡架构
    * 分布式相关
  * 平台相关
    * ELK 日志管理
    * Zabbix 分布式监控
    * 自动化运维平台
    * CMDB 平台
  * 刁钻问题
    * 秒杀&高并发场景
    * 浏览器回车
    * 海量服务器发布回滚流程设计
    * 多机房服务器帐号统一管理
    * 解决DDos攻击
    * 跨机房容灾
* [Zabbix](zabbix.md)
  * 常见问题
    * [中文乱码](zhong-wen-luan-ma.md)
    * [监控项nodata](jiankong-xiang-nodata.md)
  * 安装
    * [快速安装](zabbix/kuai-su-an-zhuang.md)
      * [Docker方式](zabbix/kuai-su-an-zhuang/dockerfang-shi.md)
  * 表结构
    * [监控表结构](jian-kong-biao-jie-gou.md)
    * [存储表结构](cun-chu-biao-jie-gou.md)
    * [报警表结构](bao-jing-biao-jie-gou.md)
  * 自发现
  * 主动注册
  * [告警依赖](zabbix/gao-jing-yi-lai.md)
  * 告警通知

