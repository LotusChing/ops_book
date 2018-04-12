# 系列1 2核心4G

## 第一轮
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
    total time:                          32.7356s
    total number of events:              10000
    total time taken by event execution: 32.7327
    per-request statistics:
         min:                                  3.22ms
         avg:                                  3.27ms
         max:                                  3.86ms
         approx.  95 percentile:               3.29ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   32.7327/0.00
```

## 第二轮
```
sysbench --test=cpu --cpu-max-prime=20000 run
sysbench 0.4.12:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 1

Doing CPU performance benchmark

Threads started!
Done.

Maximum prime number checked in CPU test: 20000


Test execution summary:
    total time:                          32.7054s
    total number of events:              10000
    total time taken by event execution: 32.7024
    per-request statistics:
         min:                                  3.25ms
         avg:                                  3.27ms
         max:                                  4.14ms
         approx.  95 percentile:               3.29ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   32.7024/0.00
```

## 第三轮
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
    total time:                          32.7046s
    total number of events:              10000
    total time taken by event execution: 32.7022
    per-request statistics:
         min:                                  3.24ms
         avg:                                  3.27ms
         max:                                  3.84ms
         approx.  95 percentile:               3.29ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   32.7022/0.00
```

## 结果

3轮测试：**32s左右**