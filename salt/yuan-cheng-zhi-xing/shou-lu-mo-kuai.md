# 手撸模块

## 为啥要写一个自己写模块？

当有些功能比较常用时，可以单独拎出来写一个模块

## 在那些模块？
```
mkdir /srv/salt/_modules
```

## 怎么写模块？
```
root@server:/srv/salt/_modules# cat my_disk.py 
def list():
    cmd = 'df -h'
    ret = __salt__['cmd.run'](cmd)
    return ret
```

## 写完就能用了么？
开发自定义模块和开发自定义_grains一样，都需要手动刷新，其实也就是把master端写的模块文件同步文件到minion端，位置是一样的，都是在minion端的/var/cache/salt/files/下面
```
salt '*' saltutil.sync_modules
```

## 怎么执行模块方法？
```
root@server:/srv/salt/_modules# salt '*' my_disk.list 
server:
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda1       902G  183G  674G  22% /
    none            4.0K     0  4.0K   0% /sys/fs/cgroup
    udev            7.9G  4.0K  7.9G   1% /dev
    tmpfs           1.6G  2.0M  1.6G   1% /run
    none            5.0M     0  5.0M   0% /run/lock
    none            7.9G  5.7M  7.9G   1% /run/shm
    none            100M     0  100M   0% /run/user
iZ947mgy3c5Z:
    Filesystem      Size  Used Avail Use% Mounted on
    udev            486M  4.0K  486M   1% /dev
    tmpfs           100M  636K   99M   1% /run
    /dev/xvda1       40G   34G  4.0G  90% /
    none            4.0K     0  4.0K   0% /sys/fs/cgroup
    none            5.0M     0  5.0M   0% /run/lock
    none            497M  1.6M  495M   1% /run/shm
    none            100M     0  100M   0% /run/user
    none             40G   34G  4.0G  90% /var/lib/docker/aufs/mnt/c6b8cdc1e53317ede04f748b183694435e52e7739b31902f8dc1ecdf574870ff
    none             40G   34G  4.0G  90% /var/lib/docker/aufs/mnt/a3c302028d4f383883aa4c8a745ca938088285357fc72fa82c78a250eccc8dd3
    none             40G   34G  4.0G  90% /var/lib/docker/aufs/mnt/54b24a249d7d6fcb8468c7e7c61c4004a9849cbdebbdd8a9cfca13a2917721ee
    shm              64M     0   64M   0% /var/lib/docker/containers/fd5ae877a49c6c0eb50557ee925ab54ce21d61882a28543f12305f4f4180566b/shm
    shm              64M     0   64M   0% /var/lib/docker/containers/7e2c27035515b82c8ba2620aa9323b2dfc02617c5af19d032209ff11d246a43d/shm
    shm              64M     0   64M   0% /var/lib/docker/containers/9c24ed37b466101f96e37220c049990e91f2603cc6b0e4269dc1fcfc98b96e35/shm
    none             40G   34G  4.0G  90% /var/lib/docker/aufs/mnt/e5259b1c55ac2b41e0761ec513125f9dab94e077f9b3f425b43bbc0fa08d9c1c
    shm              64M     0   64M   0% /var/lib/docker/containers/edc9ae2d3759937a0b613dc1ae068e3e98a27a6ff47f6c1f86b0c8c95aed4d21/shm
    /dev/xvdb        20G  543M   20G   3% /data
    none             40G   34G  4.0G  90% /var/lib/docker/aufs/mnt/e6c5b62999b86a2bb28887ecb7ea73a4f0ea1b00fd443140611c412d83d388eb
    shm              64M     0   64M   0% /var/lib/docker/containers/69c4d2b9df97a75949d51b517d35e62a86f8f9621d83f7628774b90222c7840e/shm
```