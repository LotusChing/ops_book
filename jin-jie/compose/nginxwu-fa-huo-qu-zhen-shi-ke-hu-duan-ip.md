# 无法获取真实客户端IP

在用Compose启动nginx时，发现IP都是来自本机的网关，查了下资料，很多人都遇到了同样的问题，但是可惜是没有什么比较正经的解决办法，很多都是黑科技，目前唯一靠谱点儿的就是用docker run --network=host启动，--network可以多个

Issue的地址
* https://github.com/moby/moby/issues/15086
