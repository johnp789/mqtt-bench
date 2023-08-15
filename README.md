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
$ docker stats --no-stream # vernemq
CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O   PIDS
6032cf137f45   mosquitto   103.64%   3.074MiB / 31.29GiB   0.01%     1.03GB / 409MB   0B / 0B     1
```

## FlashMQ
[FlashMQ](https://github.com/halfgaar/FlashMQ) can make use of multiple CPU threads
and is written in C++.  It turned in the fastest score in my testing.

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
$ docker stats --no-stream # vernemq
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
$ docker stats --no-stream # vernemq
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O        BLOCK I/O   PIDS
3b7c759f8560   aedes     101.65%   91.26MiB / 31.29GiB   0.28%     190MB / 76MB   0B / 0B     7
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

