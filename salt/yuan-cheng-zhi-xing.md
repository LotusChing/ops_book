# 远程执行

## 一条远程执行命令里都包含了什么？

这是一条简单的远程执行命令

```
root@server:~# salt '*' cmd.run 'free -m'
server:
                 total       used       free     shared    buffers     cached
    Mem:         16004      15655        348         13        203       6368
    -/+ buffers/cache:       9084       6919
    Swap:        16336       2921      13415
iZ947mgy3c5Z:
                 total       used       free     shared    buffers     cached
    Mem:           992        932         60          2         64         74
    -/+ buffers/cache:        792        199
    Swap:            0          0          0
```

命令其中包含了几个部分

* salt：命令
* '*'：执行目标
* cmd.run：调用模块中的方法
* 'free -m'：方法参数
