# 系列3 2核4G(共享计算)

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
    total time:                          25.9304s
    total number of events:              10000
    total time taken by event execution: 25.9288
    per-request statistics:
         min:                                  2.58ms
         avg:                                  2.59ms
         max:                                  3.03ms
         approx.  95 percentile:               2.60ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   25.9288/0.00
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
    total time:                          26.3289s
    total number of events:              10000
    total time taken by event execution: 26.3271
    per-request statistics:
         min:                                  2.58ms
         avg:                                  2.63ms
         max:                                  3.87ms
         approx.  95 percentile:               2.90ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   26.3271/0.00
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
    total time:                          26.6798s
    total number of events:              10000
    total time taken by event execution: 26.6778
    per-request statistics:
         min:                                  2.58ms
         avg:                                  2.67ms
         max:                                  3.52ms
         approx.  95 percentile:               3.03ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   26.6778/0.00
```

## 结果
**26s~**