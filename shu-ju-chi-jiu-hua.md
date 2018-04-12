# 数据持久化

## 数据持久化
## 使用Volume持久化
manager创建Service
```docker
root@node-40:~# docker service create --replicas 2 --mount 'type=volume,src=test,dst=/data' --name v-web nginx:1.13
root@node-40:~# docker service ls
y0og90z7ny3v        v-web               replicated          2/2                 nginx:1.13       
```

查看volume的详细信息
```docker
root@node-50:~# docker volume inspect test
[
    {
        "CreatedAt": "2018-02-26T10:08:20+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/test/_data",
        "Name": "test",
        "Options": {},
        "Scope": "local"
    }
]
```


worker检查
```docerk
root@node-50:~# ls /var/lib/docker/volumes/test
_data
root@node-50:~# > /var/lib/docker/volumes/test/123
root@node-50:~# docker exec -it v-web.2.w6sfteuwvieblesv701xwfa2t ls /data
123
```



## 使用NFS持久化
```docker
root@node-40:~# apt-get install nfs-kernel-server nfs-common
root@node-40:~# cat /etc/exports
/prodata/www/web 192.168.2.0/24(rw)
root@node-40:~# cat /prodata/www/web/index.html 
Index.
```


创建Service应用NFS存储，这里遇到了一个问题，manager和worker的主机上的Dcoker版本不一直，导致的问题，已记录到问题2
```docker
root@node-40:/var/lib/docker/volumes# docker service create --replicas 2 --mount 'type=volume,src=nfs-test,dst=/usr/share/nginx/html,volume-driver=local,volume-opt=type=nfs,volume-opt=device=192.168.2.40:/prodata/www/web,"volume-opt=o=addr=192.168.2.40,vers=4,soft,timeo=180,bg,tcp,rw"' -p 8881:80 --name nfs_web nginx:1.12
s01wnq6uo6jnrlkkk3s3918we
overall progress: 0 out of 2 tasks 
overall progress: 2 out of 2 tasks 
1/2: running   [==================================================>] 
2/2: running   [==================================================>] 
verify: Service converged 
```


检查文件挂在情况
```docker
root@node-40:/var/lib/docker/volumes# docker exec -it nfs_web.1.cmdxngosc9a5r6u4z02o9vwih ls /usr/share/nginx/html
index.html
```

测试访问查看
```docker
root@node-40:/var/lib/docker/volumes# curl 127.0.0.1:8881
Index.
root@node-40:/var/lib/docker/volumes# echo "LotusChing" >> /prodata/www/web/index.html 
root@node-50:~# curl 127.0.0.1:8881
Index.
LotusChing
```
