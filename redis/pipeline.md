# Pipeline

Redis通过Pipeline实现在一次网络传输中执行多个Redis命令，执行结果也会按照原先的顺序返回回来

## Pipeline原理
原先Redis执行顺序：

1. 客户端发送命令cmd1
2. 服务端Redis接收命令cmd1
3. 服务端将命令cmd1加入Redis队列
4. 服务端执行命令cmd1
5. 服务端cmd1返回结果
6. 客户端接收cmd1结果

cmd2..
cmd3..
cmd4..

如果是多条命令每次都是着走很多个来回，而使用Pipeline则将多条命令的发送和结果的接收在一次就处理完

Pipeline执行顺序：
1. 客户端发送命令cmd1、2、3、4的pipeline格式序列
2. 服务端Redis接收cmd1、2、3、4的Pipeline序列
3. 服务端将pipeline序列按顺序加入Redis队列
4. 服务端顺序执行命令，cmd1、2、3、4
5. 服务端组织执行结果，cmd1、2、3、4
6. 服务端返回cmd1、2、3、4的执行结果，也是pipeline格式序列
7. 客户端接收pipeline结果序列


**在网络延迟较大的环境下，Pipeline效果越发明显**

## Python 使用 Pipeline(待补充)

## Pipeline和原生批量命令对比
* 原生批量命令是原子的，pipeline不是，什么是原子性？
* 原生批量命令是一个命令对应一个或多个KEY，pipeline则是一个或多个命令
* 原生批量命令是服务端实现，pipeline需要客户端和服务端共同实现

## 最佳实践(待补充)
1. 虽然Pipeline能够较少网络传输上的消耗，但是如果太大量的命令封装在一个pipeline里有可能会导致Redis阻塞，建议将大的pipeline分拆成一个个小的pipeline，然后顺序执行
2. Redis只能操作一个实例，具体在分布式中如何使用pipeline还是个问题
