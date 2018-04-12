# SSD

阿里云SSD如果想实现最大性能的话，需要具备以下几个条件
* 2核4G以上
* IO实例优化
* SSD容量不要太少(不然也就比高效云盘好上一点点)


## 测试随机写IOPS
### 测试命令
```
# fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
```
### 测试结果
**iops=6008**
```
Starting 1 process
Rand_Write_Testing: Laying out IO file(s) (1 file(s) / 1024MB) 
Jobs: 1 (f=1): [w] [100.0% done] [0KB/24060KB/0KB /s] [0/6015/0 iops] [eta 00m:00s]
Rand_Write_Testing: (groupid=0, jobs=1): err= 0: pid=17333: Mon Jul 17 09:15:08 2017
  write: io=1024.0MB, bw=24036KB/s, iops=6008, runt= 43626msec
    slat (usec): min=4, max=72714, avg=14.18, stdev=400.70
    clat (usec): min=409, max=135752, avg=21286.02, stdev=33948.06
     lat (usec): min=422, max=135763, avg=21300.32, stdev=33948.91
    clat percentiles (usec):
     |  1.00th=[ 1496],  5.00th=[ 2008], 10.00th=[ 2288], 20.00th=[ 2704],
     | 30.00th=[ 3024], 40.00th=[ 3376], 50.00th=[ 3728], 60.00th=[ 4192],
     | 70.00th=[ 4896], 80.00th=[83456], 90.00th=[86528], 95.00th=[87552],
     | 99.00th=[88576], 99.50th=[89600], 99.90th=[98816], 99.95th=[102912],
     | 99.99th=[118272]
    bw (KB  /s): min=23088, max=24736, per=99.91%, avg=24013.33, stdev=183.74
    lat (usec) : 500=0.01%, 750=0.08%, 1000=0.13%
    lat (msec) : 2=4.60%, 4=51.74%, 10=20.91%, 20=0.73%, 50=0.54%
    lat (msec) : 100=21.17%, 250=0.08%
  cpu          : usr=1.01%, sys=7.44%, ctx=29796, majf=0, minf=7
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued    : total=r=0/w=262144/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
  WRITE: io=1024.0MB, aggrb=24035KB/s, minb=24035KB/s, maxb=24035KB/s, mint=43626msec, maxt=43626msec

Disk stats (read/write):
  vda: ios=0/261067, merge=0/4560, ticks=0/5515268, in_queue=5516836, util=99.81%
```


## 测试随机读IOPS
### 测试命令
```
# fio -direct=1 -iodepth=128 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```
### 测试结果
**iops=6008**
```
Starting 1 process
Jobs: 1 (f=1): [r] [100.0% done] [24004KB/0KB/0KB /s] [6001/0/0 iops] [eta 00m:00s]
Rand_Read_Testing: (groupid=0, jobs=1): err= 0: pid=17489: Mon Jul 17 09:18:52 2017
  read : io=1024.0MB, bw=24036KB/s, iops=6008, runt= 43626msec
    slat (usec): min=1, max=88048, avg= 5.16, stdev=172.69
    clat (usec): min=400, max=186606, avg=21294.72, stdev=35538.72
     lat (usec): min=408, max=186609, avg=21300.01, stdev=35538.85
    clat percentiles (usec):
     |  1.00th=[ 1192],  5.00th=[ 1624], 10.00th=[ 1864], 20.00th=[ 2224],
     | 30.00th=[ 2512], 40.00th=[ 2768], 50.00th=[ 3024], 60.00th=[ 3312],
     | 70.00th=[ 3696], 80.00th=[87552], 90.00th=[89600], 95.00th=[90624],
     | 99.00th=[91648], 99.50th=[91648], 99.90th=[93696], 99.95th=[95744],
     | 99.99th=[100864]
    bw (KB  /s): min=23368, max=24640, per=99.92%, avg=24016.64, stdev=240.94
    lat (usec) : 500=0.02%, 750=0.13%, 1000=0.31%
    lat (msec) : 2=13.07%, 4=60.56%, 10=4.37%, 20=0.21%, 50=0.05%
    lat (msec) : 100=21.27%, 250=0.01%
  cpu          : usr=1.11%, sys=3.60%, ctx=62918, majf=0, minf=137
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued    : total=r=262144/w=0/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
   READ: io=1024.0MB, aggrb=24035KB/s, minb=24035KB/s, maxb=24035KB/s, mint=43626msec, maxt=43626msec

Disk stats (read/write):
  vda: ios=261001/29, merge=0/13, ticks=5550196/236, in_queue=5552880, util=99.85%
```

## 测试写吞吐量
### 测试命令
```
# fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=64k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Write_PPS_Testing
```
### 测试结果
**149797KB/s**
```
Starting 1 process
Jobs: 1 (f=1): [W] [100.0% done] [0KB/153.3MB/0KB /s] [0/2452/0 iops] [eta 00m:00s]
Write_PPS_Testing: (groupid=0, jobs=1): err= 0: pid=17508: Mon Jul 17 09:21:22 2017
  write: io=1024.0MB, bw=149797KB/s, iops=2340, runt=  7000msec
    slat (usec): min=4, max=3873, avg=12.95, stdev=31.19
    clat (msec): min=1, max=240, avg=27.32, stdev=21.54
     lat (msec): min=1, max=240, avg=27.33, stdev=21.54
    clat percentiles (msec):
     |  1.00th=[    7],  5.00th=[   10], 10.00th=[   12], 20.00th=[   15],
     | 30.00th=[   16], 40.00th=[   18], 50.00th=[   19], 60.00th=[   21],
     | 70.00th=[   25], 80.00th=[   49], 90.00th=[   58], 95.00th=[   62],
     | 99.00th=[   69], 99.50th=[   74], 99.90th=[  241], 99.95th=[  241],
     | 99.99th=[  241]
    bw (KB  /s): min=106990, max=159872, per=100.00%, avg=150223.92, stdev=13406.16
    lat (msec) : 2=0.01%, 4=0.20%, 10=5.62%, 20=49.76%, 50=26.02%
    lat (msec) : 100=18.01%, 250=0.39%
  cpu          : usr=1.31%, sys=2.51%, ctx=924, majf=0, minf=9
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.2%, >=64=99.6%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued    : total=r=0/w=16384/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
  WRITE: io=1024.0MB, aggrb=149796KB/s, minb=149796KB/s, maxb=149796KB/s, mint=7000msec, maxt=7000msec

Disk stats (read/write):
  vda: ios=0/16159, merge=0/2, ticks=0/436700, in_queue=437132, util=98.64%
```

## 测试读吞吐量
### 测试命令
```
# fio -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=64k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Read_PPS_Testing
```
### 测试结果
**153728KB/s**
```
Starting 1 process
Jobs: 1 (f=1): [R] [100.0% done] [150.0MB/0KB/0KB /s] [2400/0/0 iops] [eta 00m:00s]
Read_PPS_Testing: (groupid=0, jobs=1): err= 0: pid=17521: Mon Jul 17 09:22:42 2017
  read : io=1024.0MB, bw=153728KB/s, iops=2401, runt=  6821msec
    slat (usec): min=6, max=604, avg=13.41, stdev= 6.59
    clat (msec): min=1, max=83, avg=26.62, stdev=29.48
     lat (msec): min=1, max=83, avg=26.63, stdev=29.48
    clat percentiles (usec):
     |  1.00th=[ 2672],  5.00th=[ 6560], 10.00th=[ 7456], 20.00th=[ 8160],
     | 30.00th=[ 8384], 40.00th=[ 8768], 50.00th=[ 9152], 60.00th=[ 9792],
     | 70.00th=[12864], 80.00th=[74240], 90.00th=[77312], 95.00th=[79360],
     | 99.00th=[81408], 99.50th=[81408], 99.90th=[82432], 99.95th=[82432],
     | 99.99th=[83456]
    bw (KB  /s): min=149120, max=158208, per=99.95%, avg=153646.85, stdev=2478.60
    lat (msec) : 2=0.40%, 4=1.74%, 10=59.57%, 20=9.97%, 50=2.29%
    lat (msec) : 100=26.03%
  cpu          : usr=0.94%, sys=5.28%, ctx=15811, majf=0, minf=522
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.2%, >=64=99.6%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued    : total=r=16384/w=0/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
   READ: io=1024.0MB, aggrb=153727KB/s, minb=153727KB/s, maxb=153727KB/s, mint=6821msec, maxt=6821msec

Disk stats (read/write):
  vda: ios=16078/12, merge=0/5, ticks=423876/220, in_queue=425616, util=98.62%
```
