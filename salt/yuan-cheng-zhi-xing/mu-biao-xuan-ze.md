# 目标选择

* **目标管理最重要的思想就是，让具体的操作执行到对应minion上**

* **不要本末倒置，会再多的匹配规则不是目的，简单，完成目的即可**


## 有哪些中方式可以帮助目标选择？

分为两种
 1. 与minion_id相关，例如server*
 2. 与minion_id无关，例如利用grains，pillar
 

### 基于minion_id
#### 猜下一下命令会输出那么minion的内存信息？

1. 匹配所有minion
 ```
 salt '*' cmd.run 'free -m'
 ```

2. 匹配i开头的minion
 ```
 salt 'i*' cmd.run 'free -m'
 ```

3. 匹配minion_id中数字范围，PS: []中只能有一位数字
 ```
 salt 'iZ947mgy[3-4]c5Z' cmd.run 'free -m'
 ```
 
4. 匹配minion中多个数字
 **错误写法**
 ```
 salt 'iZ[947|948]mgy3c5Z' cmd.run 'free -m'
 No minions matched the target. No command was sent, no jid was assigned.
ERROR: No return received

 ```
 正确写法
 ```
 salt 'iZ[9|8][4|3][7|6]mgy3c5Z' cmd.run 'free -m'
 iZ947mgy3c5Z:
                 total       used       free     shared    buffers     cached
    Mem:           992        923         69          2         41         78
    -/+ buffers/cache:        803        188
    Swap:            0          0          0
 ```
 
5. 取反操作，minion_id中不以i开头的minion
 ```
 salt '[!i]*' cmd.run 'free -m'
 ```
 
6. **列表操作**，这个对于搞自动化运维来说绝对有用，使用`salt —L`参数
 ```
 salt -L 'server, iZ947mgy3c5Z' cmd.run 'free -m'
 ```

7. 正则表达式，比原生的统配符匹配灵活很多，使用`salt -E`参数
 ```
 salt -E '(^s|^i)' cmd.run 'free -m'
 ```
 ```
 salt -E '^s.*r$' cmd.run 'free -m'
 ```
 
#### 如何控制根据minion_id来匹配minion的复杂度？
**用心设计hostname**
 1. 三思而后行，minion_id不好改，尤其是在大规模下
 2. 以业务为基准
 3. 即便当前业务规模并不是很大，但是设计之初就要考虑到以后多机房和多集群和多业务线的问题


**例子**：`redis-node1-redis04-bjidc01-soa-example.com`

这个主机名最好从后往前看
* **example.com**：公司域名，内部dns解析
* **soa**：业务线
* **bjidc01**：北京idc编号为1的机房
* **redis04**：集群名
* **redis-node1**：第一个节点


所以连起来就是，example.com公司->soa业务线->北京idc01机房->redis04集群->redis-node1节点


### 基于IP和子网
salt支持通过网段和ip地址来进行目标选择，使用`salt -S`参数
基于ip
 ```
 salt -S '192.168.1.150' test.ping
 ```
基于网段
 ```
 salt -S '192.168.1.0/24' test.ping
 ```

### 基于分组
不怎么感冒，因为需要重启salt-master，简单记下写法和用法
修改/etc/salt/master
```
1137 nodegroups:
1138   web: 'L@server,iZ947mgy3c5Z'
```

使用
```
salt -N web test.ping
```

### 如何用salt来避免批量重启服务，短暂时间内服务不可用的情况？(分批处理)
答案是分批执行，salt支持执行数量和百分比进行执行，先执行10%，再执行百分之10...，直至全部完成


**指定执行数量单位**：按照1台为单位进行执行
```
root@server:~# salt '*' -b 1 test.ping

Executing run on ['server']

jid:
    20170929103817498846
retcode:
    0
server:
    True

Executing run on ['iZ947mgy3c5Z']

iZ947mgy3c5Z:
    True
jid:
    20170929103817649095
retcode:
    0
```

**指定百分比执行单位**：
```
root@server:~# salt -G 'os:Ubuntu' --batch-size 50% test.ping

Executing run on ['server']

jid:
    20170929103956878418
retcode:
    0
server:
    True

Executing run on ['iZ947mgy3c5Z']

iZ947mgy3c5Z:
    True
jid:
    20170929103957025968
retcode:
    0
```

## Salt支持的目标选择的参数有哪些？
！不写了，每次复习时思考下！