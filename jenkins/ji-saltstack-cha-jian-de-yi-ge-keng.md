# 记saltstack插件的一个坑

## 环境
* Jenkins：2.7.4
* SaltStack: 2017.7.1
* SaltStack Plugin：3.1.4、3.1.6

## 问题现象
当使用local_batch作为客户端接口实现滚动执行时，jenkins中salt插件请求salt-api返回500错误

## 问题定位
**1. 手动调用salt-api**
通过命令行使用curl调salt-api测试

登录获取token
```bash
# curl -sSk 'http://ip/login' -H 'Accept: application/x-yaml' -d username=user -d password=pass -d eauth=pam
```

调用salt-api执行命令
```bash
# curl -sSk http://ip/ -H 'Accept: application/x-yaml' -H 'X-Auth-Token:c5bb0001df331ac0f9fa4123141b39fed2eb441a' -d client=local_batch -d tgt='server1,server2' -d tgt_type='list' -d batch=50% -d fun=cmd.run -d arg="whoami"
```

**2. 问题分析**
测试后发现通过命令行是可以的，但是通过jenkins中的salt插件就是不行

所以问题很大可能跟salt-api没关，于是抓包分析salt插件发出的http请求，以及命令行的请求有什么不同

最终发现抓包后发现两者的请求参数有些差异，salt插件的参数是用expr_form字段标识类型的
```json
{"client":"local_batch","tgt":"server,iZ947mgy3c5Z","expr_form":"list","batch":"50%","fun":"cmd.run","arg":"whoami"}
```
网上查询下有关expr_form参数，发现这个参数不支持的，应该用tgt_type参数替代它，下面是原话

>The target type should be passed using the 'tgt_type' argument instead of 'expr_form'. Support for using 'expr_form' will be removed in Salt Fluorine.


**3. 确认问题**
为了进一步确认问题，我把salt插件hpi文件下载下来，然后解压hpi，salt插件主要就是这个jar
```bash
➜  salt ls
help-blockbuild.html  help-minionTimeout.html  META-INF  WEB-INF
➜  salt ls -al WEB-INF/lib/saltstack.jar 
-rw-r--r-- 1 root root 68995 Mar 16 19:25 WEB-INF/lib/saltstack.jar
```

通过procyon-decompiler反编译后，定位到发出http请求的那个java文件
```bash
# java -jar procyon-decompiler-0.5.30.jar -o . saltstack.jar
# grep -R -n --color "expr_form" ./*                               
./com/waytta/SaltAPIBuilder.java:337:                saltFunc.put("expr_form", (Object)this.getTargettype());
./com/waytta/SaltAPIBuilder.java:346:                saltFunc.put("expr_form", (Object)this.getTargettype());
./com/waytta/SaltAPIBuilder.java:367:                saltFunc.put("expr_form", (Object)this.getTargettype());
```

本来到这就以为离成功就差一步了，但是还是碰到了小坎儿，如果将反编译的代码编译回去，擦~   这有点恶心了，我很久没有用javac编译过java代码了，现在都是用maven，搞的优点忘了javac该怎么用了，硬着头皮搞了半天，一直报错


## 问题解决

最终，忽然想到这插件是开源的，那github上肯定有源代码啊，于是到jenkins插件首页，在salt插件的详情中找到了项目的地址，最终果然找到了，https://github.com/jenkinsci/saltstack-plugin.git，看了下SaltAPIBuilder.java这个文件，果然是expr_form

接下来就比较简单了，拉代码，改代码，修改maven配置，执行mvn install编译打包

拉、改代码
```
# git clone https://github.com/jenkinsci/saltstack-plugin.git
# sed -r -i s/expr_form/tgt_type/g src/main/java/com/waytta/SaltAPIBuilder.java
```

修改maven配置，添加jenkins插件相关
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <localRepository>/root/.m2/repository</localRepository>
  <pluginGroups>
    <pluginGroup>org.jenkins-ci.tools</pluginGroup> 
  </pluginGroups>
  <profiles>
    <profile>
      <id>jenkins</id>
      <activation>
        <activeByDefault>true</activeByDefault> 
      </activation>
      <repositories> 
        <repository>
          <id>repo.jenkins-ci.org</id>
          <url>https://repo.jenkins-ci.org/public/</url>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>repo.jenkins-ci.org</id>
          <url>https://repo.jenkins-ci.org/public/</url>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
  <mirrors>
  </mirrors>
</settings>
```

编译打包，顺利的话，会在target中看到saltstack.hpi，下载下来，然后通过jenkins管理插件上传安装即可
```bash
# mvn install -DskipTests
```




