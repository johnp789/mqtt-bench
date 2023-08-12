# Playing with MQTT brokers

## Mosquitto

```
$ ./mqtt-benchmark --broker tcp://localhost:1883 --clients 1000 --qos 1 --count 500 -message-interval 0 -quiet | tail
========= TOTAL (1000) =========
Total Ratio:                 1.000 (500000/500000)
Total Runtime (sec):         5.002
Average Runtime (sec):       4.957
Msg time min (ms):           0.091
Msg time max (ms):           129.119
Msg time mean mean (ms):     9.733
Msg time mean std (ms):      0.065
Average Bandwidth (msg/sec): 100.867
Total Bandwidth (msg/sec):   100866.723
```

With a longer `--count 5000` test, the docker stats show
```
$ docker stats --no-stream # vernemq
CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O   PIDS
6032cf137f45   mosquitto   103.64%   3.074MiB / 31.29GiB   0.01%     1.03GB / 409MB   0B / 0B     1
```

## FlashMQ

```
$ ./mqtt-benchmark --broker tcp://localhost:1883 --clients 1000 --qos 1 --count 500 -message-interval 0 -quiet | tail
========= TOTAL (1000) =========
Total Ratio:                 1.000 (500000/500000)
Total Runtime (sec):         1.568
Average Runtime (sec):       1.494
Msg time min (ms):           0.036
Msg time max (ms):           153.230
Msg time mean mean (ms):     2.705
Msg time mean std (ms):      0.115
Average Bandwidth (msg/sec): 334.997
Total Bandwidth (msg/sec):   334996.982
```

With a longer `--count 5000` test, the docker stats show
```
$ docker stats --no-stream # vernemq
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O   PIDS
e52533fe40e9   flashmq   393.64%   32.49MiB / 31.29GiB   0.10%     1.56GB / 616MB   0B / 0B     27
```

## VerneMQ
```
$ ./mqtt-benchmark --broker tcp://localhost:1883 --clients 1000 --qos 1 --count 500 -message-interval 0 -quiet | tail
========= TOTAL (1000) =========
Total Ratio:                 1.000 (500000/500000)
Total Runtime (sec):         2.659
Average Runtime (sec):       2.568
Msg time min (ms):           0.069
Msg time max (ms):           120.965
Msg time mean mean (ms):     4.730
Msg time mean std (ms):      0.108
Average Bandwidth (msg/sec): 195.047
Total Bandwidth (msg/sec):   195046.868
```

With a longer `--count 5000` test, the docker stats show
```
$ docker stats --no-stream # vernemq
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O          BLOCK I/O   PIDS
4c595202c1eb   vernemq   792.02%   462MiB / 31.29GiB   1.44%     1.95GB / 770MB   0B / 0B     219
```

## Aedes
```
$ ./mqtt-benchmark --broker tcp://localhost:1883 --clients 1000 --qos 1 --count 500 -message-interval 0 -quiet | tail
========= TOTAL (1000) =========
Total Ratio:                 1.000 (500000/500000)
Total Runtime (sec):         15.008
Average Runtime (sec):       14.948
Msg time min (ms):           2.218
Msg time max (ms):           218.754
Msg time mean mean (ms):     29.008
Msg time mean std (ms):      0.155
Average Bandwidth (msg/sec): 33.452
Total Bandwidth (msg/sec):   33451.684
```

With a longer `--count 5000` test, the docker stats show
```
$ docker stats --no-stream # vernemq
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O        BLOCK I/O   PIDS
3b7c759f8560   aedes     101.65%   91.26MiB / 31.29GiB   0.28%     190MB / 76MB   0B / 0B     7
```

