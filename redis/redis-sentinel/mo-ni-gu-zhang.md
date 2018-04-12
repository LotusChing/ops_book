# 模拟故障

## kill -9 redis_pid

Sentinel节点1宕机日志
```
2236:X 05 Jan 17:30:50.758 # +sdown master mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:50.825 # +odown master mymaster 192.168.2.50 6379 #quorum 2/2
2236:X 05 Jan 17:30:50.825 # +new-epoch 4
2236:X 05 Jan 17:30:50.825 # +try-failover master mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:50.828 # +vote-for-leader 4aa9c09483ec5fd8ffcce32159bc76168e8e4104 4
2236:X 05 Jan 17:30:50.835 # 192.168.2.30:26379 voted for 4aa9c09483ec5fd8ffcce32159bc76168e8e4104 4
2236:X 05 Jan 17:30:50.836 # 192.168.2.50:26379 voted for 4aa9c09483ec5fd8ffcce32159bc76168e8e4104 4
2236:X 05 Jan 17:30:50.895 # +elected-leader master mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:50.895 # +failover-state-select-slave master mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:50.986 # +selected-slave slave 192.168.2.30:6379 192.168.2.30 6379 @ mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:50.986 * +failover-state-send-slaveof-noone slave 192.168.2.30:6379 192.168.2.30 6379 @ mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:51.039 * +failover-state-wait-promotion slave 192.168.2.30:6379 192.168.2.30 6379 @ mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:52.028 # +promoted-slave slave 192.168.2.30:6379 192.168.2.30 6379 @ mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:52.028 # +failover-state-reconf-slaves master mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:52.030 * +slave-reconf-sent slave 192.168.2.20:6379 192.168.2.20 6379 @ mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:52.977 * +slave-reconf-inprog slave 192.168.2.20:6379 192.168.2.20 6379 @ mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:52.977 * +slave-reconf-done slave 192.168.2.20:6379 192.168.2.20 6379 @ mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:53.047 # -odown master mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:53.047 # +failover-end master mymaster 192.168.2.50 6379
2236:X 05 Jan 17:30:53.047 # +switch-master mymaster 192.168.2.50 6379 192.168.2.30 6379
2236:X 05 Jan 17:30:53.048 * +slave slave 192.168.2.20:6379 192.168.2.20 6379 @ mymaster 192.168.2.30 6379
2236:X 05 Jan 17:30:53.048 * +slave slave 192.168.2.50:6379 192.168.2.50 6379 @ mymaster 192.168.2.30 6379
```

Sentinel节点2宕机日志
```
[root@Da redis]# tail -f data/26379.log 
31724:X 10 Jan 10:51:38.196 # +sdown master mymaster 192.168.2.50 6379
31724:X 10 Jan 10:51:38.267 # +new-epoch 4
31724:X 10 Jan 10:51:38.269 # +vote-for-leader 4aa9c09483ec5fd8ffcce32159bc76168e8e4104 4
31724:X 10 Jan 10:51:38.280 # +odown master mymaster 192.168.2.50 6379 #quorum 3/2
31724:X 10 Jan 10:51:38.280 # Next failover delay: I will not start a failover before Wed Jan 10 10:57:39 2018
31724:X 10 Jan 10:51:39.467 # +config-update-from sentinel 192.168.2.20:26379 192.168.2.20 26379 @ mymaster 192.168.2.50 6379
31724:X 10 Jan 10:51:39.467 # +switch-master mymaster 192.168.2.50 6379 192.168.2.30 6379
31724:X 10 Jan 10:51:39.468 * +slave slave 192.168.2.20:6379 192.168.2.20 6379 @ mymaster 192.168.2.30 6379
31724:X 10 Jan 10:51:39.468 * +slave slave 192.168.2.50:6379 192.168.2.50 6379 @ mymaster 192.168.2.30 6379
31724:X 10 Jan 10:52:09.470 # +sdown slave 192.168.2.50:6379 192.168.2.50 6379 @ mymaster 192.168.2.30 6379
```


Sentinel节点3宕机日志
```
5592:X 10 Jan 10:51:38.164 # +sdown master mymaster 192.168.2.50 6379
5592:X 10 Jan 10:51:38.276 # +new-epoch 4
5592:X 10 Jan 10:51:38.278 # +vote-for-leader 4aa9c09483ec5fd8ffcce32159bc76168e8e4104 4
5592:X 10 Jan 10:51:39.233 # +odown master mymaster 192.168.2.50 6379 #quorum 3/2
5592:X 10 Jan 10:51:39.234 # Next failover delay: I will not start a failover before Wed Jan 10 10:57:39 2018
5592:X 10 Jan 10:51:39.476 # +config-update-from sentinel 192.168.2.20:26379 192.168.2.20 26379 @ mymaster 192.168.2.50 6379
5592:X 10 Jan 10:51:39.478 # +switch-master mymaster 192.168.2.50 6379 192.168.2.30 6379
5592:X 10 Jan 10:51:39.479 * +slave slave 192.168.2.20:6379 192.168.2.20 6379 @ mymaster 192.168.2.30 6379
5592:X 10 Jan 10:51:39.481 * +slave slave 192.168.2.50:6379 192.168.2.50 6379 @ mymaster 192.168.2.30 6379
5592:X 10 Jan 10:52:09.518 # +sdown slave 192.168.2.50:6379 192.168.2.50 6379 @ mymaster 192.168.2.30 6379
```