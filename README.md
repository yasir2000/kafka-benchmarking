# kafka-benchmarking

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
