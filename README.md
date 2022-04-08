# Kafka benchmarking

Scripts to run a performance test against a Kafka cluster.  
Uses the kafka-producer-perf-test/kafka-consumer-perf-test scripts, which are included in Kafka deplyoments.  
In the OpenSource Kafka tgz, these scripts have a suffix ```.sh```, whereas in the Confluent distribution they don't....just be aware of that ;)

* ```benchmark-producer.sh``` : execute one producer benchmark run with a dedicated set of properties
* ```benchmark-consumer.sh``` : execute one consumer benchmark run with a dedicated set of properties
* ```benchmark-suite-producer.sh``` : wrapper around _benchmark-producer.sh_ to execute multiple benchmark runs with varying property settings

The output of the benchmark execution will be stored within a .txt file in the same directory as the benchmark-*.sh scripts are, if parameter ```--output-to-file``` is specified only.
Repeating benchmark executions with the same properties will append the output to existing output file.

## Prerequisites

* a running Kafka cluster
* a host which has the Kafka client tools installed (scripts _kafka-topics_, _kafka-producer-perf-test_, _kafka-consumer-perf-test_, ...)
* tools _readlink_ and _tee_ installed
* ensure that your Kafka cluster has enough free space to maintain the data which is being created during the benchmark run(s)

## Single benchmark execution

### Producer benchmark

**NOTE**
> the scripts for running producer benchmark(s) you'll find in directory [**_producer/scripts_**](./producer/scripts/)


A single benchmark execution for a Producer can be executed via calling ```benchmark-producer.sh``` directly, providing commandline parameters. You can set environment variables pointing to the executables for ```kafka-topics```-command as well as for ```kafka-producer-perf-test```-command as shown below.  

* Environment variables
  
  | variable | description | example
  | -------- | ----------- | -------
  | KAFKA_TOPICS_CMD | how to execute the kafka-topics command (full path or relative) | /opt/kafka/bin/kafka-topics.sh
  | KAFKA_BENCHMARK_CMD | how to execute the kafka-producer-perf-test command (full path or relative) | /opt/kafka/bin/kafka-producer-perf-test-sh

* Parameters are:
  
  | parameter | description | default |
  | --------- | ----------- | ------- |
  | -p \| --partitions _\<number\>_  | where _\<number\>_ is an int, telling how many partitions the benchmark topic shall have. **Only required if you want the script manage the topic for the benchmark** | 2
  | -r \| --replicas _\<number\>_  | where _\<number\>_ is an int, telling how many replicas the benchmark topic shall have. **OOnly required if you want the script manage the topic for the benchmark** | 2  
  | --num-records _\<number\>_ |  where _\<number\>_ specifies how many messages shall be created during the benchmark. | 100000  
  | --record-size _\<number\>_ |  where _\<number\>_ specifies how big (in bytes) each record shall be. | 1024  
  | --producer-props _<string\>_ | list of additional properties for the benchmark execution, like e.g. ```acks```, ```linger.ms```, ... | 'acks=1 compression.type=none'
  | --bootstrap-servers _\<string\>_ | comma separated list of \<host\>:\<port\> of your Kafka brokers. This property is **mandatory** |
  | --throughput _\<string\>_ | specifies the throughput to use during the benchmark run | -1
  | --topic _\<string\>_ |  specifies the topic to use for the benchmark execution. This topic **must** exist before you execute this script.  |
  | --producer-config _\<config-file\>_ | config-file to provide additional attributes to connect to broker(s), mainly **SSL** & **authentication** |

* Output
  
  The script generates 2 output files  
  * .txt: **one file per execution**, this file includes all messages printed during the performance test execution
  * .csv: **one file per day**, this file includes the pure metrics, comma separated, and commented the start-/finish-time as well as the parameters for the test execution
  

**NOTE**

> Topic management for the execution:    
> If you don't have a pre-existing topic and you don't want to manage the topic by yourself, just **omit** property ```--topic```. 
> By that the script will create a topic before the benchmark run, and delete it afterwards.  
> If you already have a topic you want to use for the benchmark execution, then provide its name via ```--topic```

---
**Usage examples**

* run benchmark with minimal parameters, use the existing topic _bench-topic_ on local Kafka broker with port 9091:
  
  ```bash
  ./benchmark-producer.sh --bootstrap-servers localhost:9091 --topic bench-topic
  ```

* run benchmark with minimal parameters, let the script manage the topic on local Kafka broker with port 9091:
  
  ```bash
  ./benchmark-producer.sh --bootstrap-servers localhost:9091 --partitions 5 --replicas 2
  ```

* tuning for **Throughput**:
  
  ```bash
  ./benchmark-producer.sh --bootstrap-servers localhost:9091 --partitions 3 --replicas 2 --record-size 1024 --num-records 200000 --producer-props 'acks=1 compression.type=lz4 batch.size=100000 linger.ms=50'
  ```

* tuning for **Latency**:
  
  ```bash
  ./benchmark-producer.sh --bootstrap-servers localhost:9091 --partitions 3 --replicas 2 --record-size 1024 --num-records 200000 --producer-props 'acks=1 compression.type=lz4 batch.size=10000 linger.ms=0'
  ```

* tuning for **Durability**
  
  ```bash
  ./benchmark-producer.sh --bootstrap-servers localhost:9091 --partitions 3 --replicas 2 --record-size 1024 --num-records 200000 --producer-props 'acks=1 compression.type=lz4 batch.size=10000 linger.ms=0'
  ```

* run benchmark with minimal parameters, let the script manage the topic on local Kafka broker with port 9092 and provide producer.config including SASL_PLAINTEXT info:
  
  ```bash
  ./benchmark-producer.sh --bootstrap-servers localhost:9091 --partitions 5 --replicas 2 --producer-config ./sample-producer-sasl.config
  ```
  where content of _sample-producer-sasl.config_ for a PLAINTEXT SASL auth (user + password) can be:  
  ```
  security.protocol=SASL_PLAINTEXT
  sasl.mechanism=PLAIN
  sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="admin-secret";
  ```

---

### Consumer benchmark

**NOTE**
> the scripts for running consumer benchmark(s) you'll find in directory [**_consumer/scripts_**](./consumer/scripts/)

A single benchmark execution for a Consumer can be executed via calling ```benchmark-consumer.sh``` directly, providing commandline parameters.  
Parameters are:
 | parameter | description | default |
 | --------- | ----------- | ------- |
 | --topic _\<string\>_ |  specifies the topic to use for the benchmark execution. This property is **mandatory** |
 | --bootstrap-server _\<string\>_ | comma separated list of \<host\>:\<port\> of your Kafka brokers.  | localhost:9091
 | --messages _\<number\>_ |  where _\<number\>_ specifies how many messages shall be consumed during the benchmark. | 10000 
 | --fetch-max-wait-ms _\<number\>_  | specifies the timeout the server waits to collect _fetch-min-bytes_ to return to the client ([official doc](https://kafka.apache.org/documentation/#consumerconfigs_fetch.max.wait.ms)) | 500
 | --fetch-min-bytes _\<number\>_  | The minimum amount of data the server should return for a fetch request. <br/>Value of "1" means _do not wait and send data as soon as there is some_ ([official doc](https://kafka.apache.org/documentation/#consumerconfigs_fetch.min.bytes)) | 1  
 | --fetch-size  |  The amount of data to fetch in a single request | 1048576
 | --enable-auto-commit _\<number\>_ |  If true the consumer's offset will be periodically committed in the background | true  
 | --isolation-level _<string\>_ | specifies how transactional message are being read ([official doc](https://kafka.apache.org/documentation/#consumerconfigs_isolation.level)) | read_uncommitted
 | --consumer-config _\<config-file\>_ | config-file to provide additional attributes to connect to broker(s), mainly **SSL** & **authentication** |
 | --group-id _\<string\>_ | the ConsumerGroup id for the consumer. <br/>**Important** if you have ACLs enabled, to specify permissions for this group as well | consumer-benchmark
 | --verbose | if specified, additional text output will be printed to the terminal |
  
**Usage examples**

* run consumer benchmark with minimal arguments, using kafka broker on localhost:9091 and topicname _my-benchmark-topic_:
  
  ```./benchmark-consumer.sh --topic my-benchmark-topic```

* providing an config file containing e.g. properties for SASL authentication (as shown in [sasl-properties.config](consumer/scripts/sasl-properties.config)):

  ```./benchmark-consumer.sh --topic my-benchmark-topic --consumer-config ./sasl-properties.config```

* same as above, but with some more output on the commandline:

  ```./benchmark-consumer.sh --topic my-benchmark-topic --consumer-config ./sasl-properties.config --verbose```

* tuning for **Throughput**:

  ```./benchmark-consumer.sh --topic my-benchmark-topic --fetch-min-bytes 100000```

* tuning for **Latency**:

  ```./benchmark-consumer.sh --topic my-benchmark-topic --fetch-size 25000 --fetch-max-wait-ms 100```

## Batch of benchmark executions

Script ```benchmark-suite-producer.sh``` is just a wrapper around _benchmark-producer.sh_ to run a variety of performance test runs against your Kafka cluster. It loops over the properties you want to change between test runs and calls _benchmark-producer.sh_ once for each single combination of properties.  
The properties, which are possible to iterate over, you'll find within ```benchmark-suite-producer.sh```, section  **variables**.  

The only **mandatory** parameter is: ```--bootstrap-servers``` , the bootstrap server(s) to connect to as comma separated list ```--bootstrap-servers <host>:<port>```

Parameters to adjust for your UseCase, in ```benchmark-suite-producer.sh```, are:

| Parameter | Description | Example
| --------- | ----------- | -------
PARTITIONS | space separated list of number of partitions for the benchmark topic | PARTITIONS="2 10"
REPLICAS | space separated list of number of replicas for the benchmark topic | REPLICAS="2"
NUM_RECORDS | space separated list of number of records to produce | NUM_RECORDS="100000"
RECORD_SIZES | space separated list of record sizes | RECORD_SIZES="1024 10240"
THROUGHPUT | space separated list of desired throughput limits, "-1" means: no limit, full speed | THROUGHPUT="-1"
ACKS | space separated list of values for the producer property "acks", valid values "0 1 -1" | ACKS="0 -1"
COMPRESSION | compression type to use, e.g. "none","lz4",... | COMPRESSION="none"
LINGER_MS | space separated list of desired values for "linger.ms" kafka property | LINGER_MS="0"
BATCH_SIZE | space separated list of desired values for "batch.size" kafka property, "0": disable batching | BATCH_SIZE="10000"

# Docker

to be able to run the benchmarks within a container, you can use the provided Dockerfile to create such a container.  
Alternatively you'll find prebuilt docker container on Docker Hub:
- Producer Image: [gkoenig/kafka-producer-benchmark](https://hub.docker.com/repository/docker/gkoenig/kafka-producer-benchmark)
- Consumer Image: [gkoenig/kafka-consumer-benchmark](https://hub.docker.com/repository/docker/gkoenig/kafka-consumer-benchmark)

**NOTE**

> - you have to mount a local directory to the container path _/tmp/output_ so that you can save the output files from the benchmark run on your local workstation.
> - ensure that the host directory (which you mount into the container) has permissions, so that the container can write file(s) to it

## Producer benchmark usage examples

To be able to store the output files from the benchmark run on your local workstation/laptop, create a local dir and mount it into the container. Otherwise you won't be able to access the files after the benchmark run is finished.

### minimal example

the following example shows a scenario with minimal parameters. It will run a **producer** benchmark, connecting to Kafka (unauthenticated) on port 9091 on IP _000.111.222.333_ **you obviously have to provide you IP address of your Kafka broker here !!!**
```
mkdir ./output
chmod 777 ./output
docker run  -v ./output/:/tmp/output gkoenig/kafka-producer-benchmark:0.1 --bootstrap-servers 000.111.222.333:9091
```

### with authentication

if you have a kafka cluster with authentication, then you can provide additional parameters as you would do on a _plain_ execution of benchmark-producer.sh script, e.g. if your brokers listen on port 9092 for SASL_PLAINTEXT authentication, then specify the port and provide additional producer-config as below (**you obviously have to provide you IP address of your Kafka broker here !!!**):  

```
git clone git@github.com:gkoenig/kafka-benchmarking.git
cd producer/scripts
chmod 777 ./output
docker run  -v ./output/:/tmp/output gkoenig/kafka-producer-benchmark:0.1 --bootstrap-servers 000.111.222.333:9092 --producer-config sample-producer-sasl.config
```

### providing more parameters

You can provide the same list of parameters as if you execute the producer-benchmark outside of Docker containers, means plain on the terminal.
Detailled description of parameters can be found [here](https://github.com/gkoenig/kafka-benchmarking#producer-benchmark)

```
git clone git@github.com:gkoenig/kafka-benchmarking.git
cd scripts
chmod 777 ./output
docker run  -v ./output/:/tmp/output gkoenig/kafka-producer-benchmark:0.1 --bootstrap-servers 000.111.222.333:9092 --producer-config sample-producer-sasl.config --num-records 2000000 --compression lz4
```

# Kubernetes

## Description

To run the producer benchmark container within K8s cluster, you need to ensure the following prereq's:
- you need to have the possibility to access a persistent volume (to store the output of the benchmark run), mounted on _/tmp/output_ within the container
- **!! only if you have to specify SASL properties to connect to your Kafka brokers**
to be able to pass the SASL properties, you need to create a ConfigMap from the file sample-producer-sasl.config (or whichever file you created including your SASL configuration to talk to the Kafka brokers) and use that ConfigMap as a volume in the container spec. **Ensure** that the mountpoint of this ConfigMap corresponds with the _args_ property specifying the _--producer-config_ parameter (see example below).
If you can connect to your brokers unauthenticated, you do not need this configmap, and you have to delete the corresponding volumeMount in the Job yaml as well.

## ConfigMap

```bash
# assuming you are in the root folder of the Github repo you cloned
cd producer
kubectl create configmap kafka-sasl --from-file=scripts/sample-producer-sasl.config 
```

## Job specification

To execute the benchmark it is sufficient to create a Kubernetes Job, that
- mounts the persistent volume to store the output
- mounts the configmap, including the SASL config **!! of course only, if you have to specify SASL properties to connect to your Kafka brokers**

Ensure that you replace the IP (or hostname) and port of your Kafka broker inside the [producer-benchmark-job.yaml](./producer/producer-benchmark-job.yaml). The placeholder is set to _111.222.333.444:9092_

```bash
# assuming you are in the root folder of the Github repo you cloned
# if the job already exists, you first have to delete it, before it is created again
cd producer
# kubectl delete -f ./producer-benchmark-job.yaml
kubectl apply -f ./producer-benchmark-job.yaml
```
Producer

Setup
bin/kafka-topics.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --create --topic test-rep-one --partitions 6 --replication-factor 1
bin/kafka-topics.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --create --topic test --partitions 6 --replication-factor 3

Single thread, no replication

bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test7 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196

Single-thread, async 3x replication

bin/kafktopics.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --create --topic test --partitions 6 --replication-factor 3
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test6 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196

Single-thread, sync 3x replication

bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000 100 -1 acks=-1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=64000

Three Producers, 3x async replication
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196

Throughput Versus Stored Data

bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196

Effect of message size

for i in 10 100 1000 10000 100000;
do
echo ""
echo $i
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test $((1000*1024*1024/$i)) $i -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=128000
done;

#############################################################################################
Consumer
Consumer throughput

bin/kafka-consumer-perf-test.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --messages 50000000 --topic test --threads 1

3 Consumers

On three servers, run:
bin/kafka-consumer-perf-test.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --messages 50000000 --topic test --threads 1

End-to-end Latency

bin/kafka-run-class.sh kafka.tools.TestEndToEndLatency esv4-hcl198.grid.linkedin.com:9092 esv4-hcl197.grid.linkedin.com:2181 test 5000

Producer and consumer

bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196

bin/kafka-consumer-perf-test.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --messages 50000000 --topic test --threads 1


server-config.properties


############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0

############################# Socket Server Settings #############################

# The port the socket server listens on
port=9092

# Hostname the broker will bind to and advertise to producers and consumers.
# If not set, the server will bind to all interfaces and advertise the value returned from
# from java.net.InetAddress.getCanonicalHostName().
#host.name=localhost

# The number of threads handling network requests
num.network.threads=4
 
# The number of threads doing disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=1048576

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=1048576

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# The directory under which to store log files
log.dirs=/grid/a/dfs-data/kafka-logs,/grid/b/dfs-data/kafka-logs,/grid/c/dfs-data/kafka-logs,/grid/d/dfs-data/kafka-logs,/grid/e/dfs-data/kafka-logs,/grid/f/dfs-data/kafka-logs

# The number of logical partitions per topic per server. More partitions allow greater parallelism
# for consumption, but also mean more files.
num.partitions=8

############################# Log Flush Policy #############################

# The following configurations control the flush of data to disk. This is the most
# important performance knob in kafka.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data is at greater risk of loss in the event of a crash.
#    2. Latency: Data is not made available to consumers until it is flushed (which adds latency).
#    3. Throughput: The flush is generally the most expensive operation. 
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# Per-topic overrides for log.flush.interval.ms
#log.flush.intervals.ms.per.topic=topic1:1000, topic2:3000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log as long as the remaining
# segments don't drop below log.retention.bytes.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=536870912

# The interval at which log segments are checked to see if they can be deleted according 
# to the retention policies
log.cleanup.interval.mins=1

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=esv4-hcl197.grid.linkedin.com:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=1000000

# metrics reporter properties
kafka.metrics.polling.interval.secs=5
kafka.metrics.reporters=kafka.metrics.KafkaCSVMetricsReporter
kafka.csv.metrics.dir=/tmp/kafka_metrics
# Disable csv reporting by default.
kafka.csv.metrics.reporter.enabled=false

replica.lag.max.messages=10000000

#############################################################
targetRate - op/s, expected target throughput
warmupTime - sec, test warmup time
runTime - sec, test run time
intervalLength - ms, histogram writer interval
reset - call reset before each benchmark run by 'Runner'
histogramsDir - location for histogram files
brokerList - list of Kafka brokers
topic - Kafka topic used for testing
partitions - Kafka topic partitions number used in the topic creation inside benchmark reset() if BenchmarkConfig.reset is true
replicationFactor - Kafka topic replication factor used in the topic creation inside benchmark reset() if BenchmarkConfig.reset is true
createTopicRetries - number of topic creation retries
waitAfterDeleteTopic - seconds, time to do nothing after topic deletion
messageLength - minimum size of a message in bytes
messageLengthMax - maximum size of a message in bytes if > messageLength, else = messageLength
producerThreads - number of producer threads 
consumerThreads - number of consumer threads
pollTimeout - consumer poll timeout 
producerAcks - ProducerConfig.ACKS_DOC
batchSize - ProducerConfig.BATCH_SIZE_CONFIG  

###################################################################

Producer
Setup

bin/kafka-topics.sh \
  --zookeeper zookeeper.example.com:2181 \
  --create \
  --topic test-rep-one \
  --partitions 6 \
  --replication-factor 1
bin/kafka-topics.sh \
  --zookeeper zookeeper.example.com:2181 \
  --create \
  --topic test \
  --partitions 6 --replication-factor 3
Single thread, no replication

bin/kafka-producer-perf-test.sh \
  --topic test \
  --num-records 50000000 \
  --record-size 100 \
  --throughput -1 \
  --producer-props acks=1 \
  bootstrap.servers=kafka.example.com:9092 \
  buffer.memory=67108864 \
  batch.size=8196
Single-thread, async 3x replication

bin/kafk-topics.sh \
  --zookeeper zookeeper.example.com:2181 \
  --create \
  --topic test \
  --partitions 6 \
  --replication-factor 3
bin/kafka-producer-perf-test.sh \
  --topic test \
  --num-records 50000000 \
  --record-size 100 \
  --throughput -1 \
  --producer-props acks=1 \
  bootstrap.servers=kafka.example.com:9092 \
  buffer.memory=67108864 \
  batch.size=8196
Single-thread, sync 3x replication

bin/kafka-producer-perf-test.sh \
  --topic test \
  --num-records 50000000 \
  --record-size 100 \
  --throughput -1 \
  --producer-props acks=1 \
  bootstrap.servers=kafka.example.com:9092 \
  buffer.memory=67108864 batch.size=64000
Three Producers, 3x async replication

bin/kafka-producer-perf-test.sh \
  --topic test \
  --num-records 50000000 \
  --record-size 100 \
  --throughput -1 \
  --producer-props acks=1 \
  bootstrap.servers=kafka.example.com:9092 \
  buffer.memory=67108864 \
  batch.size=8196
Throughput Versus Stored Data

bin/kafka-producer-perf-test.sh \
  --topic test \
  --num-records 50000000 \
  --record-size 100 \
  --throughput -1 \
  --producer-props acks=1 \
  bootstrap.servers=kafka.example.com:9092 \
  buffer.memory=67108864 batch.size=8196
Effect of message size

for i in 10 100 1000 10000 100000; do
  echo ""
  echo $i
  bin/kafka-producer-perf-test.sh \
    --topic test \
    --num-records $((1000*1024*1024/$i))\
    --record-size $i\
    --throughput -1 \
    --producer-props acks=1 \
    bootstrap.servers=kafka.example.com:9092 \
    buffer.memory=67108864 \
    batch.size=128000
done;
Consumer
Consumer throughput

bin/kafka-consumer-perf-test.sh \
  --zookeeper zookeeper.example.com:2181 \
  --messages 50000000 \
  --topic test \
  --threads 1
3 Consumers

On three servers, run:

bin/kafka-consumer-perf-test.sh \
  --zookeeper zookeeper.example.com:2181 \
  --messages 50000000 \
  --topic test \
  --threads 1
End-to-end Latency
bin/kafka-run-class.sh \
  kafka.tools.TestEndToEndLatency \
  kafka.example.com:9092 \
  zookeeper.example.com:2181 \
  test 5000
Producer and consumer
bin/kafka-run-class.sh \
  org.apache.kafka.tools.ProducerPerformance \
bin/kafka-producer-perf-test.sh \
  --topic test \
  --num-records 50000000 \
  --record-size 100 \
  --throughput -1 \
  --producer-props acks=1 \
  bootstrap.servers=kafka.example.com:9092 \
  buffer.memory=67108864 \
  batch.size=8196
bin/kafka-consumer-perf-test.sh \
  --zookeeper zookeeper.example.com:2181 \
  --messages 50000000 \
  --topic test \
  --threads 1