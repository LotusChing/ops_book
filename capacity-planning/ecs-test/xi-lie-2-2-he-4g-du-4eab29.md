# 系列2 2核4G(独享)

> Intel Xeon E5-2680 v3（Haswell）处理器，2.5GHz 的主频。
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
    total time:                          25.9009s
    total number of events:              10000
    total time taken by event execution: 25.8997
    per-request statistics:
         min:                                  2.58ms
         avg:                                  2.59ms
         max:                                  3.78ms
         approx.  95 percentile:               2.59ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   25.8997/0.00
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
    total time:                          25.9593s
    total number of events:              10000
    total time taken by event execution: 25.9580
    per-request statistics:
         min:                                  2.58ms
         avg:                                  2.60ms
         max:                                  3.77ms
         approx.  95 percentile:               2.61ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   25.9580/0.00
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
    total time:                          25.8912s
    total number of events:              10000
    total time taken by event execution: 25.8900
    per-request statistics:
         min:                                  2.58ms
         avg:                                  2.59ms
         max:                                  2.70ms
         approx.  95 percentile:               2.59ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   25.8900/0.00
```

## 结果
**26s~**