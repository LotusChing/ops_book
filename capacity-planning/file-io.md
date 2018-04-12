# File IO

Sysbench同样支持文件IO测试，所以我们继续来测试，这里IO时混合读写，因为比较接近我们的真实情况

## 测试
这里要注意一下，`file-total-size`要设置大于内存大小，不然会缓存
```
# sysbench --test=fileio --file-total-size=2G --file-test-mode=rndrw --init-rng=on --max-time=300 --max-requests=0 run
sysbench 0.4.12:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 1
Initializing random number generator from timer.


Extra file open flags: 0
128 files, 16Mb each
2Gb total file size
Block size 16Kb
Number of random requests for random IO: 0
Read/Write ratio for combined random IO test: 1.50
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Threads started!
Time limit exceeded, exiting...
Done.

Operations performed:  16680 Read, 11120 Write, 35541 Other = 63341 Total
Read 260.62Mb  Written 173.75Mb  Total transferred 434.38Mb  (1.4478Mb/sec)
   92.66 Requests/sec executed

Test execution summary:
    total time:                          300.0148s
    total number of events:              27800
    total time taken by event execution: 226.8928
    per-request statistics:
         min:                                  0.01ms
         avg:                                  8.16ms
         max:                                423.57ms
         approx.  95 percentile:              19.01ms

Threads fairness:
    events (avg/stddev):           27800.0000/0.00
    execution time (avg/stddev):   226.8928/0.00
```



测试结果里比较重要的是这行，可以看到`1.4478Mb`是**每秒读写的速度**
```
Read 260.62Mb  Written 173.75Mb  Total transferred 434.38Mb  (1.4478Mb/sec)
```


