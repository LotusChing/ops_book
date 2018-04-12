# CPU

使用`Sysbench`工具对`CPU`计算能力进行测试

## 安装
### Ubuntu 
```
# apt-get -y install sysbench
```

### Centos
```
# yum -y install sysbench
```

## 测试
```
# sysbench --test=cpu --cpu-max-prime=20000 run
sysbench 0.4.12:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 1
　
Doing CPU performance benchmark
　
Threads started!
Done.
　
Maximum prime number checked in CPU test: 20000
　
　
Test execution summary:
    total time:                          30.1030s
    total number of events:              10000
    total time taken by event execution: 30.1011
    per-request statistics:
         min:                                  3.00ms
         avg:                                  3.01ms
         max:                                  3.38ms
         approx.  95 percentile:               3.02ms
　
Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   30.1011/0.00
``` 

其中最为关键的就是`execution time`，这是总的计算时间，越短越好



