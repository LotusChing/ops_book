# 系列3 2核8G(共享通用)

> Intel Xeon E5-2682 v4 (Broadwell) 处理器，2.5GHz 的主频。

## 第一轮
```
sysbench 0.4.12:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 1

Doing CPU performance benchmark

Threads started!
Done.

Maximum prime number checked in CPU test: 20000


Test execution summary:
    total time:                          25.9093s
    total number of events:              10000
    total time taken by event execution: 25.9081
    per-request statistics:
         min:                                  2.58ms
         avg:                                  2.59ms
         max:                                  3.64ms
         approx.  95 percentile:               2.59ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   25.9081/0.00
```

## 第二轮
```
sysbench 0.4.12:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 1

Doing CPU performance benchmark

Threads started!
Done.

Maximum prime number checked in CPU test: 20000


Test execution summary:
    total time:                          25.8885s
    total number of events:              10000
    total time taken by event execution: 25.8872
    per-request statistics:
         min:                                  2.58ms
         avg:                                  2.59ms
         max:                                  2.65ms
         approx.  95 percentile:               2.59ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   25.8872/0.00
```

## 第三轮
```
sysbench 0.4.12:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 1

Doing CPU performance benchmark

Threads started!
Done.

Maximum prime number checked in CPU test: 20000


Test execution summary:
    total time:                          25.9042s
    total number of events:              10000
    total time taken by event execution: 25.9030
    per-request statistics:
         min:                                  2.58ms
         avg:                                  2.59ms
         max:                                  2.72ms
         approx.  95 percentile:               2.59ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   25.9030/0.00
```

## 结果
**26~**