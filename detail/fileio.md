# 文件读写

## IO
首先，要明确一个事情，这里提到的`Directory`并不是通常说的目录，而是记录文件和目录名称对应关系的列表
1. 根据文件名通过`Directory`里找对应关系，找到文件对应的`inode number`
2. 根据`inode number`读取文件对应的`inode table`信息(`atime`、`ctime`、`mtime`、`uid`、`gid`、`size`、`ts`、`permission`、`link count`以及**Pointer**等等)
3. 根据`Inode table`里的`Pointer`读取到文件对应的`blocks`
4. 一个`inode`对应一个文件，但是个文件会根据自身大小，有可能会占用多个`block`
5. 硬链接的原理就是在`Directory`中多写入了新的对应关系而已
6. 删除文件本质上调用`unlink`就是减少`link count`，当`link count`等于`0`时，就表示这个`inode`可以使用，并把对应的`blocks`标记为可写，此时并不清除对应的`block`中的数据，直到有新的数据需要用到这些`blocks`才会被覆盖
7. 移动文件分为两种情况
  * 同一文件系统中：
   1. 将新文件名(包括路径)和`inode`的对应关系写入`Directory`
   2. 删除`Directory`中旧的对应关系
   3. 更新`ctime`
  * 不同文件系统中：
   1. 找一个空的`inode`，然后写入新的`inode table`信息
   2. 在`Directory`中创建新的对应关系
   3. 然后向`blocks`写入文件内容
   4. 修改`ctime`
   
   
   
之前公司测试环境里Tomcat的ROOT目录总是诡异的消失，后来通过audit来捕捉针对ROOT目录的操作，就发现了其中指定了一个叫做unlinkat的系统调用，当时半天没反应过来，现在明白了，不过目录消失的原因仍然没有找到，无语~

通过strace rm可以看到unlinkat的细节

```
root@server:/tmp# strace rm -f dbbackup_2017-02-07.gz 
execve("/bin/rm", ["rm", "-f", "dbbackup_2017-02-07.gz"], [/* 23 vars */]) = 0
brk(0)                                  = 0x1602000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f2433de7000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=44380, ...}) = 0
mmap(NULL, 44380, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f2433ddc000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P \2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=1840928, ...}) = 0
mmap(NULL, 3949248, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f2433802000
mprotect(0x7f24339bc000, 2097152, PROT_NONE) = 0
mmap(0x7f2433bbc000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1ba000) = 0x7f2433bbc000
mmap(0x7f2433bc2000, 17088, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f2433bc2000
close(3)                                = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f2433ddb000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f2433dd9000
arch_prctl(ARCH_SET_FS, 0x7f2433dd9740) = 0
mprotect(0x7f2433bbc000, 16384, PROT_READ) = 0
mprotect(0x60d000, 4096, PROT_READ)     = 0
mprotect(0x7f2433de9000, 4096, PROT_READ) = 0
munmap(0x7f2433ddc000, 44380)           = 0
brk(0)                                  = 0x1602000
brk(0x1623000)                          = 0x1623000
open("/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=1607664, ...}) = 0
mmap(NULL, 1607664, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f2433c50000
close(3)                                = 0
ioctl(0, SNDCTL_TMR_TIMEBASE or SNDRV_TIMER_IOCTL_NEXT_DEVICE or TCGETS, {B38400 opost isig icanon echo ...}) = 0
newfstatat(AT_FDCWD, "dbbackup_2017-02-07.gz", {st_mode=S_IFREG|0644, st_size=171839173, ...}, AT_SYMLINK_NOFOLLOW) = 0

unlinkat(AT_FDCWD, "dbbackup_2017-02-07.gz", 0) = 0

lseek(0, 0, SEEK_CUR)                   = -1 ESPIPE (Illegal seek)
close(0)                                = 0
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```


 
