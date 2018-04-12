# 系列1 2核4G

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
    total time:                          37.5221s
    total number of events:              10000
    total time taken by event execution: 37.5196
    per-request statistics:
         min:                                  3.71ms
         avg:                                  3.75ms
         max:                                  6.50ms
         approx.  95 percentile:               3.80ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   37.5196/0.00
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
    total time:                          37.4771s
    total number of events:              10000
    total time taken by event execution: 37.4743
    per-request statistics:
         min:                                  3.70ms
         avg:                                  3.75ms
         max:                                  5.60ms
         approx.  95 percentile:               3.77ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   37.4743/0.00
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
    total time:                          37.4626s
    total number of events:              10000
    total time taken by event execution: 37.4601
    per-request statistics:
         min:                                  3.71ms
         avg:                                  3.75ms
         max:                                  7.24ms
         approx.  95 percentile:               3.76ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   37.4601/0.00
```
## 结果
**37s**
