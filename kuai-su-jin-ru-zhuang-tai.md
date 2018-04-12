# 快速开始

内容来自简书的**[灼灼2015](https://www.jianshu.com/u/dc22c7608c73)**小姐姐的两篇文章，感谢，自己整理下方便理解吸收

* [Linux中级实用--快速搭建环境](https://www.jianshu.com/p/4360e0436e3b)
* [Linux运维实用--逻辑部署图](https://www.jianshu.com/p/919be5ca051e)



## 基础环境：
	1. 开发语言和版本
	java、php、python、go、ruby

	2. 根据开发语言来判断用什么容器和版本
	tomcat、apache、nginx、tornado、flask

	3. 确定项目用的什么数据库和版本
	oracle、mysql、redis、mongodb

	4. 确定项目组成结构
	如是拆分多个的，如后台和前台，需要知道数据流的走向。（可以问研发）
	还有当前项目是否依赖其他的第三方接口，例如天气、用以监控时确认问题来源
	5. 确定项目是否用到第三方软件和版本
	rabbitmq、kafka

	6. 确定项目要运行什么操作系统和版本
	如centos、ubuntu、windows
	

## 网络环境：
 * 物理机房：机房网络环境拓扑图
 * 云提供商：VPC、安全组拓扑图


## 数据流向：
1. 数据从哪里来？
2. 数据存储在哪里？（数据库or硬盘）
3. 数据到哪里去？


## 数据量：
 * QPS/TPS: 低峰平峰高峰业务状态(不同时间内响应时间阈值、状态码分布阈值)


## 搭建：
	各个击破，从最底层开始装，就像建房子需要地基一样。
	操作系统-->
	项目启动环境（运行容器tomcat，数据库，编译语言）-->
	打包应用-->
	建数据库、建表、导初始数据-->
	容器加载应用-->
	应用的配置-->这里最好让研发写个配置说明，数据库怎么连、日志怎么配置、目录配置用途
	配置第三方软件-->
	完成搭建