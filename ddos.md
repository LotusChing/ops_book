# DDos

## 攻击姿势
* CC攻击：攻击消耗较高系统资源的点
* 流量反射：攻击端将源IP伪造成被攻击端的ip，向大量的被欺骗的主机发送请求，被欺骗的主机将请求响应给被攻击的主机


syn半连接+流量反射

攻击端伪造IP源ip进行syn三次握手，攻击端发送第一个syn后，被攻击端根据源ip返回ask及syn包，此时被攻击端进入sync_recv状态，当服务端未收到ack确认包时，被攻击端会重发ack、syn响应包，直至超时才会将连接从未连接队列删除

通过伪造大量无效的源ip，可以让被攻击端连接队列被沾满从而无法响应正常的握手请求

解决方法：
* 缩短超时



## 防御姿势
* 绝对不要直接暴露出公网IP
* 非机密数据可以考虑使用360、百度提供的加速(反向代理)来防御
* 对于代价较高的请求，最好先做逻辑处理，例如短信验证码，请求读取数据库
