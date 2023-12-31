# Playing with MQTT brokers

It is time to pick an MQTT broker for some homelab experiments.  Just for fun, I
tried out several available MQTT brokers.  These are all excellent brokers, each
one making different trade-offs for simplicity, developer velocity, and developer
time.  My tests are also not exhaustive; they're more of a quick look at how
these brokers behave under load.

These tests were run in WSL2 Ubuntu 22.04 on an AMD Ryzen 9 5900X machine.  Each
broker was run in a Docker container.  Unless stated otherwise, I enabled
persistence for each broker, although
[the benchmark](https://github.com/krylovsk/mqtt-benchmark) did not set the
retain flag on the messages it published.

## Mosquitto

[Mosquitto](https://github.com/eclipse/mosquitto) is a classic broker written
in C.  The speed is excellent given the single CPU thread and low memory usage.

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
$ docker stats --no-stream 
CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O   PIDS
6032cf137f45   mosquitto   103.64%   3.074MiB / 31.29GiB   0.01%     1.03GB / 409MB   0B / 0B     1
```

## FlashMQ
[FlashMQ](https://github.com/halfgaar/FlashMQ) can make use of multiple CPU threads
and is written in C++.  It tied with HiveMQ-CE for the fastest score in my
testing.  It is possibly saturating the testing tool.

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
$ docker stats --no-stream 
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O   PIDS
e52533fe40e9   flashmq   393.64%   32.49MiB / 31.29GiB   0.10%     1.56GB / 616MB   0B / 0B     27
```

## VerneMQ
[VerneMQ](https://github.com/vernemq/vernemq) scales vertically and horizontally
and is written in Erlang.  While on my single 24-thread machine it scored between
Mosquitto and FlashMQ, it would likely outperform either in a cluster deployment.

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
$ docker stats --no-stream 
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O          BLOCK I/O   PIDS
4c595202c1eb   vernemq   792.02%   462MiB / 31.29GiB   1.44%     1.95GB / 770MB   0B / 0B     219
```

## Aedes
[Aedes](https://github.com/moscajs/aedes) is a JavaScript broker.  Here, I have
tested the official Docker image which uses Node.js.  Persistence was *not*
enabled in my testing.
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
$ docker stats --no-stream 
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O        BLOCK I/O   PIDS
3b7c759f8560   aedes     101.65%   91.26MiB / 31.29GiB   0.28%     190MB / 76MB   0B / 0B     7
```

## Aedes with Bun
[Aedes](https://github.com/moscajs/aedes) was run in a Docker container using
[bun](https://bun.sh) 1.0.2 instead of Node.js.  The performance on this test
was very similar with bun to that of Node.js, but with 3X the memory usage.
```
$ ./mqtt-benchmark --broker tcp://localhost:1883 --clients 1000 --qos 1 --count 500 -message-interval 0 -quiet | tail
========= TOTAL (1000) =========
Total Ratio:                 1.000 (500000/500000)
Total Runtime (sec):         15.101
Average Runtime (sec):       14.989
Msg time min (ms):           0.520
Msg time max (ms):           169.713
Msg time mean mean (ms):     29.717
Msg time mean std (ms):      0.041
Average Bandwidth (msg/sec): 33.358
Total Bandwidth (msg/sec):   33358.241
```

With a longer `--count 5000` test, the docker stats show
```
$ docker stats --no-stream
CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O   PIDS
46aa978c3aaa   aedes-bun   115.17%   274.8MiB / 31.29GiB   0.86%     587MB / 446MB   0B / 0B     11
```

## Mochi
[Mochi](https://github.com/mochi-mqtt/server) is a broker written in Go. I built
the sample `cmd/main.go` application locally using the Dockerfile provided in the
repo, so persistence was *not* enabled.  Mochi turned in a very good result,
nearly matching FlashMQ, albeit with much more CPU and memory usage.

```
$ ./mqtt-benchmark --broker tcp://localhost:1883 --clients 1000 --qos 1 --count 500 -message-interval 0 -quiet | tail
========= TOTAL (1000) =========
Total Ratio:                 1.000 (500000/500000)
Total Runtime (sec):         1.918
Average Runtime (sec):       1.792
Msg time min (ms):           0.039
Msg time max (ms):           157.679
Msg time mean mean (ms):     3.408
Msg time mean std (ms):      0.141
Average Bandwidth (msg/sec): 279.320
Total Bandwidth (msg/sec):   279319.588
```

With a longer `--count 5000` test, the docker stats show
```
$ docker stats --no-stream
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O   PIDS
78451b5cea0b   mochi     611.45%   239.7MiB / 31.29GiB   0.75%     1.24GB / 489MB   0B / 0B     39
```

## rumqttd
[rumqttd]() is less fully featured MQTT broker written in Rust: it does not
support last will, retained messages, or MQTT 5.  However, performance is good
in my little test.

```
k$ ./mqtt-benchmark --broker tcp://localhost:1883 --clients 1000 --qos 1 --count 500 -message-interval 0 -quiet | tail
========= TOTAL (1000) =========
Total Ratio:                 1.000 (500000/500000)
Total Runtime (sec):         5.954
Average Runtime (sec):       5.079
Msg time min (ms):           0.134
Msg time max (ms):           108.701
Msg time mean mean (ms):     8.220
Msg time mean std (ms):      1.320
Average Bandwidth (msg/sec): 105.642
Total Bandwidth (msg/sec):   105641.606
```

With a longer `--count 5000` test, the docker stats show
```
~$ docker stats --no-stream
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O   PIDS
12e7edd80d37   rumqttd   155.90%   100.8MiB / 31.29GiB   0.31%     517MB / 205MB   0B / 0B     6
```

## HiveMQ CE
[HiveMQ Community Edition](https://github.com/hivemq/hivemq-community-edition) is
a well-known broker written in Java.  It tied with FlashMQ for the top score,
suggesting that it may have saturated the benchmark tool.

```
$ ./mqtt-benchmark --broker tcp://localhost:1883 --clients 1000 --qos 1 --count 500 -message-interval 0 -quiet | tail
========= TOTAL (1000) =========
Total Ratio:                 1.000 (500000/500000)
Total Runtime (sec):         1.691
Average Runtime (sec):       1.482
Msg time min (ms):           0.041
Msg time max (ms):           120.646
Msg time mean mean (ms):     2.626
Msg time mean std (ms):      0.469
Average Bandwidth (msg/sec): 338.215
Total Bandwidth (msg/sec):   338215.154
```

With a longer `--count 5000` test, the docker stats show
```
$ docker stats --no-stream
CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O   PIDS
36e912734eb3   hivemq-ce   509.43%   768.5MiB / 31.29GiB   2.40%     742MB / 294MB   0B / 0B     288
```

## EMQX
[EMQX](https://github.com/emqx/emqx) is a highly scalable broker, supporting
many features and protocols in addition to MQTT.  It is written in Erlang.  In
this benchmark, it performed well, but not quite as highly as the top brokers.
CPU usage was the highest in the test so far.  The msg/sec result was very
similar to VerneMQ, the other Erlang-based broker in this roundup.

```
$ ./mqtt-benchmark --broker tcp://localhost:1883 --clients 1000 --qos 1 --count 500 -message-interval 0 -quiet | tail
========= TOTAL (1000) =========
Total Ratio:                 1.000 (500000/500000)
Total Runtime (sec):         2.711
Average Runtime (sec):       2.619
Msg time min (ms):           0.063
Msg time max (ms):           193.029
Msg time mean mean (ms):     4.980
Msg time mean std (ms):      0.128
Average Bandwidth (msg/sec): 191.105
Total Bandwidth (msg/sec):   191105.268
```

With a longer `--count 5000` test, the docker stats show
```
$ docker stats --no-stream
CONTAINER ID   NAME      CPU %      MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O   PIDS
1e94c21ee6df   emqx      1024.05%   351.9MiB / 31.29GiB   1.10%     984MB / 390MB   0B / 0B     146
```

