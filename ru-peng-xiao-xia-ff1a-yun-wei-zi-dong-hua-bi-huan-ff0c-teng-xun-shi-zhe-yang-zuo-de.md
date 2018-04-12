# 如烹小虾： 运维自动化闭环，腾讯是这样做的

原文链接：[如烹小虾： 运维自动化闭环，腾讯是这样做的](http://dockone.io/article/1303)

感谢这位前辈的分享，下面简单记录了下文章内比较有收获的内容。

## 运维自动化闭环

* 配置管理
* 状态管理
* 变更管理

## 运维的责任
> 首先，从运维的职能来看。只要干好一件事就可以，那就是让我们管的机器，或者业务能够一直正常运行，只要它不故障，基本就没有运维的事了。

> 但如果出了异常，不管什么事都会有我们的责任，这就是运维。

可能术业有专攻吧，我遇到一些开发，他们大多SQL基础不牢，而且对操作系统和基础的网络知识也不是很了解，甚至又一次在做完善监控系统的时候，我向开发收集一些业务信息，他们说不用，他们自己已经做好了，我好奇的看了下他们做的监控，每小时注册数，每小时登陆数，我看见这些指标苦笑了下，我说你这些是业务数据统计不算监控吧，举个例子，假如有用户说无法打开App或者无法使用某些模块的功能，咱们能否在用户投诉前知晓呢？

开发同事耸耸肩说我这里只要服务正常接口OK的就没问题啊，说真的我当时真的不知道说什么好，一下子尬住了，有N多种可能是在服务正常运行，网络正常，但是用户就是使用不了你的App，比如DNS解析异常、路由节点故障、服务器连接数打满、MySQL Query Hang或者磁盘空间或者磁盘IO等等等等问题，影响服务可用的因素太多太多了，看着开发同事无所谓的态度，我心里挺不是滋味的，脑子飘过了一个词，DevOps，要让自己参与更多的事情上去，开发、测试、部署、运维、监控等等，不然照这样继续下去永远背不完的锅，牢骚发的有点多了，嘿嘿~


## 运维平台建设理念
1. 不要妄图一步到位
2. 先做简单且迫切的内容(功能)，然后完善丰富
3. 设定标准化，减低兼容带来的复杂度，越标准越简单
4. 减少讨论和设计，快速出Demo，问题会在运行后自动出现
5. 接受不完美
6. 业务为导向

## 配置管理 CMDB
### ITIL
传统的ITIL理念是非常重视CMDB的，但它把所有精力都集中在硬件资源的管理，如机房、机柜、交换机、网络，这些层面的东西。ITIL下的CMDB会把这些东西管理得很好。

### DevOps
在应用程序的层面，我们可以做的事情就很多。如果能力到位了，我们甚至可以实现故障自愈、自动调度、弹性计算等高级的自动化能力。而这些能力，需要我们的CMDB能够关注和管理面向业务的配置信息。

比如说我业务资源是什么样的，我的程序是什么样的，我的应用是什么样的，我的权限是什么样的，我的流程，我的策略是什么，这些东西是非常有价值的。

只有把这些东西全面管理起来，才能够真正去驱动整个业务的自动化流程。举个例子，比如说常规的一些磁盘空间告警，我要想实现故障治愈，首先我得有明确的处理策略。

* 人工手动更新CMDB
* Agent采集汇报更新


## 变更管理
变更管理平台的实现。这里可以考虑分阶段的实现方案。对于一些基础比较差的团队来说，要想一步到位是非常困难的，而且如果能力没有达到一定的水平，有一些自动化能力也不一定可以用得起来。

先从脚本平台开始，先实现最基础的作业管理能力，之后再实现业务管理和流程管理能力。

作业管理也可以理解为脚本管理。实现自动化最简单的办法，其实就是编写脚本。每个公司的运维团队都一定会有些积累的脚本。这些都是运维人员最宝贵的资产。

通过实现作业管理平台，来提供统一的可视化脚本管理能力，它一方面能够通过分享和复用来降低脚本开发的成本，另一方面也可以实现集中控制，并保证操作的可靠性。腾讯目前的蓝鲸平台，其实本质上就是一个作业管理平台。 

如果团队能力更强一点，就可以开始考虑实现业务管理能力。通过脚本来实现自动化虽然比较简单，但面对业务管理的场景时，我们就会发现即使是一个简单的业务，都会需要大量的脚本才有可能够把业务的关联环境维护好。

比如业务发布后的进程守护、日志清理、启动初始化；
再比如业务本身的版本管理、集群管理、实例管理、回滚管理等等。


## 状态管理
说白点就是监控平台，对运维自动化来说，监控平台是一个最主要的活动触发源。如果我不知道业务运行的状态，不知道业务有没有异常，那么自动化也就无法发挥它应有的价值。 

一个好的监控系统需要能够实现端到端的监控，即全链路监控

每一个系统的最终用户都一定是人，所以我们可以从用户的角度，模拟用户的访问路径，从而实现一个全面的端到端监控能力。

我们可以把用户请求过程中所经过的节点和链路全部监控起来。再通过综合的分析和汇聚，从而有效果的掌握业务的运行状态，并及时发现异常。

具体实现时，我们可以从最底层的基础网络和链路逐步往上是应用服务器、应用组件、组件请求、服务质量、到最上层的业务状态实现全部的监控覆盖。 

### 外部探测
* ping
* nc
* traceroute
* http

### Agent上报
* 系统状态
* 业务状态
* 日志采集

监控能力对于自动平台来说，最大的价值是能够完成事件的触发。也即实现从数据发现、分析、定位、到问题解决的闭环。

通过这个闭环我们可以构建各种故障的自愈能力。通过及时的发现异常，快速的恢复，能够有效的提升业务的可用性和质量。

试想下，监控系统发现了服务挂掉了，然后通过服务所属机器的ip以及服务名到到CMDB里查询对应解决策略，是重启服务还是如何如何，执行完CMDB中修复策略后再次查看是否已修复，如果未修复则通知运维人员参与进来。

当然上面说到的只是很简单的场景，实际工作中肯定会有更多复杂的情况，不过这些问题终归是否规则的，所以只要细心琢磨并且勤于实践肯定能够减少一些重复性的工作，我想没有人喜欢三更半夜的起来解决问题吧~
