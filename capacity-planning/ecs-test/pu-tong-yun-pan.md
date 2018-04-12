# 普通云盘

## 测试随机写IOPS
```
```

测试结果
```
```

## 测试随机读IOPS
```
```

测试结果
```
```



## 测试写吞吐量
```
# fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=64k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Write_PPS_Testing
```

测试结果
```
Write_PPS_Testing: (g=0): rw=write, bs=64K-64K/64K-64K/64K-64K, ioengine=libaio, iodepth=64
fio-2.1.3
Starting 1 process
Write_PPS_Testing: Laying out IO file(s) (1 file(s) / 1024MB)
Jobs: 1 (f=1): [W] [100.0% done] [0KB/41878KB/0KB /s] [0/654/0 iops] [eta 00m:00s]
Write_PPS_Testing: (groupid=0, jobs=1): err= 0: pid=31906: Fri Jul 14 10:20:43 2017
  write: io=1024.0MB, bw=37043KB/s, iops=578, runt= 28307msec
    slat (usec): min=9, max=114784, avg=59.74, stdev=1669.48
    clat (msec): min=4, max=265, avg=110.50, stdev=30.15
     lat (msec): min=5, max=265, avg=110.56, stdev=30.11
    clat percentiles (msec):
     |  1.00th=[   33],  5.00th=[   85], 10.00th=[   91], 20.00th=[   94],
     | 30.00th=[   96], 40.00th=[   98], 50.00th=[  102], 60.00th=[  106],
     | 70.00th=[  114], 80.00th=[  130], 90.00th=[  149], 95.00th=[  172],
     | 99.00th=[  223], 99.50th=[  231], 99.90th=[  251], 99.95th=[  260],
     | 99.99th=[  265]
    bw (KB  /s): min=19175, max=47840, per=100.00%, avg=37096.44, stdev=6087.57
    lat (msec) : 10=0.01%, 50=2.08%, 100=43.28%, 250=54.53%, 500=0.11%
  cpu          : usr=0.51%, sys=1.46%, ctx=14978, majf=0, minf=25
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.2%, >=64=99.6%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued    : total=r=0/w=16384/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
  WRITE: io=1024.0MB, aggrb=37042KB/s, minb=37042KB/s, maxb=37042KB/s, mint=28307msec, maxt=28307msec

Disk stats (read/write):
  xvda: ios=385/32931, merge=313/122, ticks=8256/3677544, in_queue=3692480, util=99.75%
```

## 测试读吞吐量
```
```

测试结果
```
```





