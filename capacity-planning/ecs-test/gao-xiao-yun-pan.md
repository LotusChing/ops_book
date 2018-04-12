# 高效云盘

## 测试随机写IOPS
### 测试命令

```
# fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
```
### 测试结果
**1644 IOPS**
```
Rand_Write_Testing: (g=0): rw=randwrite, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=128
fio-2.1.3
Starting 1 process
Jobs: 1 (f=1): [w] [100.0% done] [0KB/6577KB/0KB /s] [0/1644/0 iops] [eta 00m:00s]
Rand_Write_Testing: (groupid=0, jobs=1): err= 0: pid=32212: Fri Jul 14 10:43:38 2017
  write: io=1024.0MB, bw=4568.7KB/s, iops=1142, runt=229517msec
    slat (usec): min=2, max=102984, avg=17.46, stdev=906.10
    clat (usec): min=873, max=1096.2K, avg=112046.91, stdev=68263.96
     lat (msec): min=1, max=1096, avg=112.06, stdev=68.26
    clat percentiles (msec):
     |  1.00th=[    5],  5.00th=[    8], 10.00th=[   11], 20.00th=[   90],
     | 30.00th=[   95], 40.00th=[   99], 50.00th=[  102], 60.00th=[  105],
     | 70.00th=[  111], 80.00th=[  190], 90.00th=[  196], 95.00th=[  198],
     | 99.00th=[  297], 99.50th=[  400], 99.90th=[  603], 99.95th=[  701],
     | 99.99th=[  898]
    bw (KB  /s): min= 3649, max= 6912, per=100.00%, avg=4570.27, stdev=313.01
    lat (usec) : 1000=0.01%
    lat (msec) : 2=0.01%, 4=0.44%, 10=8.75%, 20=6.05%, 50=0.50%
    lat (msec) : 100=29.17%, 250=53.57%, 500=1.27%, 750=0.21%, 1000=0.03%
    lat (msec) : 2000=0.01%
  cpu          : usr=0.72%, sys=1.59%, ctx=177199, majf=0, minf=27
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued    : total=r=0/w=262144/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
  WRITE: io=1024.0MB, aggrb=4568KB/s, minb=4568KB/s, maxb=4568KB/s, mint=229517msec, maxt=229517msec

Disk stats (read/write):
  xvdb: ios=0/256972, merge=0/5164, ticks=0/28740776, in_queue=28747580, util=100.00%
```

## 测试随机读IOPS
### 测试命令
```
# fio -direct=1 -iodepth=128 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```

### 测试结果
**1697 IOPS**
```
Rand_Read_Testing: (g=0): rw=randread, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=128
fio-2.1.3
Starting 1 process
Jobs: 1 (f=1): [r] [100.0% done] [6789KB/0KB/0KB /s] [1697/0/0 iops] [eta 00m:00s]
Rand_Read_Testing: (groupid=0, jobs=1): err= 0: pid=32253: Fri Jul 14 10:49:36 2017
  read : io=1024.0MB, bw=4570.9KB/s, iops=1142, runt=229406msec
    slat (usec): min=2, max=2089, avg= 6.85, stdev= 9.02
    clat (msec): min=1, max=596, avg=112.00, stdev=68.13
     lat (msec): min=1, max=596, avg=112.01, stdev=68.13
    clat percentiles (msec):
     |  1.00th=[    4],  5.00th=[    6], 10.00th=[    9], 20.00th=[   92],
     | 30.00th=[   96], 40.00th=[   99], 50.00th=[  101], 60.00th=[  104],
     | 70.00th=[  109], 80.00th=[  192], 90.00th=[  196], 95.00th=[  200],
     | 99.00th=[  302], 99.50th=[  400], 99.90th=[  502], 99.95th=[  506],
     | 99.99th=[  594]
    bw (KB  /s): min= 3837, max= 6992, per=100.00%, avg=4573.19, stdev=286.99
    lat (msec) : 2=0.01%, 4=1.12%, 10=11.12%, 20=3.60%, 50=0.12%
    lat (msec) : 100=29.77%, 250=52.37%, 500=1.76%, 750=0.16%
  cpu          : usr=0.69%, sys=1.44%, ctx=181526, majf=0, minf=155
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued    : total=r=262144/w=0/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
   READ: io=1024.0MB, aggrb=4570KB/s, minb=4570KB/s, maxb=4570KB/s, mint=229406msec, maxt=229406msec

Disk stats (read/write):
  xvdb: ios=256703/3, merge=5113/1, ticks=28889784/216, in_queue=28899060, util=100.00%
```

## 测试写吞吐量

### 测试命令
```
# fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=64k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Write_PPS_Testing
```
### 测试结果
**最大：56M   最小：48M**
```
Write_PPS_Testing: (g=0): rw=write, bs=64K-64K/64K-64K/64K-64K, ioengine=libaio, iodepth=64
fio-2.1.3
Starting 1 process
Jobs: 1 (f=1): [W] [100.0% done] [0KB/53322KB/0KB /s] [0/833/0 iops] [eta 00m:00s]
Write_PPS_Testing: (groupid=0, jobs=1): err= 0: pid=32277: Fri Jul 14 10:51:37 2017
  write: io=1024.0MB, bw=53325KB/s, iops=833, runt= 19664msec
    slat (usec): min=6, max=43730, avg=26.20, stdev=546.88
    clat (msec): min=4, max=132, avg=76.78, stdev=18.88
     lat (msec): min=4, max=132, avg=76.80, stdev=18.87
    clat percentiles (msec):
     |  1.00th=[   37],  5.00th=[   41], 10.00th=[   44], 20.00th=[   50],
     | 30.00th=[   80], 40.00th=[   83], 50.00th=[   85], 60.00th=[   87],
     | 70.00th=[   88], 80.00th=[   90], 90.00th=[   93], 95.00th=[   95],
     | 99.00th=[  104], 99.50th=[  115], 99.90th=[  126], 99.95th=[  128],
     | 99.99th=[  133]
    bw (KB  /s): min=48962, max=56745, per=99.86%, avg=53246.95, stdev=1469.21
    lat (msec) : 10=0.01%, 20=0.10%, 50=19.87%, 100=78.72%, 250=1.31%
  cpu          : usr=1.10%, sys=1.82%, ctx=15068, majf=0, minf=26
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.2%, >=64=99.6%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued    : total=r=0/w=16384/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
  WRITE: io=1024.0MB, aggrb=53324KB/s, minb=53324KB/s, maxb=53324KB/s, mint=19664msec, maxt=19664msec

Disk stats (read/write):
  xvdb: ios=0/32315, merge=0/3, ticks=0/2461772, in_queue=2467912, util=99.59%
```

## 测试读吞吐量
### 测试命令
```
# fio -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=64k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Read_PPS_Testing
```
### 测试结果
**读取：53M**
```
Read_PPS_Testing: (g=0): rw=read, bs=64K-64K/64K-64K/64K-64K, ioengine=libaio, iodepth=64
fio-2.1.3
Starting 1 process
Jobs: 1 (f=1): [R] [100.0% done] [52683KB/0KB/0KB /s] [823/0/0 iops] [eta 00m:00s]
Read_PPS_Testing: (groupid=0, jobs=1): err= 0: pid=32287: Fri Jul 14 10:55:16 2017
  read : io=1024.0MB, bw=53273KB/s, iops=832, runt= 19683msec
    slat (usec): min=5, max=457, avg=12.46, stdev= 9.79
    clat (msec): min=9, max=142, avg=76.86, stdev=19.16
     lat (msec): min=9, max=142, avg=76.87, stdev=19.16
    clat percentiles (msec):
     |  1.00th=[   36],  5.00th=[   39], 10.00th=[   42], 20.00th=[   53],
     | 30.00th=[   79], 40.00th=[   83], 50.00th=[   85], 60.00th=[   87],
     | 70.00th=[   89], 80.00th=[   91], 90.00th=[   94], 95.00th=[   96],
     | 99.00th=[  101], 99.50th=[  104], 99.90th=[  135], 99.95th=[  139],
     | 99.99th=[  143]
    bw (KB  /s): min=45731, max=55696, per=99.74%, avg=53135.79, stdev=1646.31
    lat (msec) : 10=0.02%, 20=0.01%, 50=18.31%, 100=80.44%, 250=1.21%
  cpu          : usr=0.37%, sys=1.99%, ctx=15887, majf=0, minf=1051
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.2%, >=64=99.6%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued    : total=r=16384/w=0/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
   READ: io=1024.0MB, aggrb=53273KB/s, minb=53273KB/s, maxb=53273KB/s, mint=19683msec, maxt=19683msec

Disk stats (read/write):
  xvdb: ios=32732/2, merge=0/1, ticks=2502344/92, in_queue=2504696, util=99.58%
```

