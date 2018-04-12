# 系列2 2核4G 共享

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
    total time:                          25.9628s
    total number of events:              10000
    total time taken by event execution: 25.9615
    per-request statistics:
         min:                                  2.59ms
         avg:                                  2.60ms
         max:                                  3.46ms
         approx.  95 percentile:               2.60ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   25.9615/0.00

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
    total time:                          25.9564s
    total number of events:              10000
    total time taken by event execution: 25.9551
    per-request statistics:
         min:                                  2.59ms
         avg:                                  2.60ms
         max:                                  2.72ms
         approx.  95 percentile:               2.60ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   25.9551/0.00
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
    total time:                          25.9495s
    total number of events:              10000
    total time taken by event execution: 25.9482
    per-request statistics:
         min:                                  2.58ms
         avg:                                  2.59ms
         max:                                  2.65ms
         approx.  95 percentile:               2.60ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   25.9482/0.00
```

## 结果
**25s**
