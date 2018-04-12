# OWASP Dependency Check

## 为什么要进行依赖检查
开发过程中不可避免的要引用一些外部的包，而这么包是否安全我不知道，但是大家默认都会认为它是安全的，但是好像并不是的，我注意到一下几个问题

"外部包"安全问题
* 包被篡改
* 包本身有安全bug

解决包安全问题
* 通过md5哈希值比对检查包是否被篡改过
* 关注开发所使用到的外部包开发组织发布的安全信息，及时更新升级或采取其他防护措施

## 用什么东西来做依赖检查
OWASP组织开发的Dependency Check就是针对Maven工程的Java项目进行依赖检查，它会统计工程中引用到包，以及各个包的版本，然后到NVD(National Vulnerability Database)国家漏洞数据库、CVE(Common Vulnerabilities and Exposures)通用漏洞与披露这两个库中搜索获取对应包的安全漏洞信息


## 怎么做依赖安全检查
### Java(Maven)
确保是Maven结构的代码，然后在pom.xml里plugins下添加一下内容
```
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>1.4.5</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

然后在工程目录下执行
```
mvn dependency-check:check
```


接下来它首先会到maven中配置库中下载引用的jar包，然后会下载NVD、CVE库(可能会比较慢)，下载完漏洞库后，就会执行依赖检查，最后在工程里的target目录下生成一份名叫dependency-check-report.html的依赖检查报告，里面会记录了工程中所使用到的外部包的安全问题

![](/assets/mvn_dependency_check_report_html.png)


## 知道问题了，如何修复安全问题
通常来说出现安全问题都是通过更新升级来解决的，但是也不排除有些时候我们无法更新，例如我们业务所使用到了某些特定版本的特定功能，或者业务系统运行周期已经很久了，担心更新升级后出现问题，像这类情况就只能根据漏洞特点自行解决了，但是大多数情况来说都是升级来解决的

### 如何升级
根据生成的安全报告，然后获取漏洞包名，然后在pom.xml找到它并复制pom.xml中对包的artifactId中的名称，然后到搜索引擎中搜索包名+maven，例如`mysql-connector-java maven`，然后新的版本号粘贴到pom.xml中对应包version属性里，然后重新执行检查并阅读报告，确认安全问题是否解决


## 补充
* OWASP Dependency Check也支持对Python的依赖包进行检查，详见[文档](https://pypi.python.org/pypi/dependency-check/)
* 不仅仅只是后端，前端也有类似的工具支持依赖检查(未完)
* 抽空把文章分开写吧