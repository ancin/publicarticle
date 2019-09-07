
# install
 Download the 2.3.0 release and un-tar it.

 tar -xzf kafka_2.12-2.3.0.tgz
> cd kafka_2.12-2.3.0

## install zk
 bin/zkService.sh start

 # start kafka

 bin/kafka-server-start.sh config/server.properties

 nohup bin/kafka-server-start.sh config/server.properties >> /dev/null &

 # create topic
  bin/kafka-topics.sh --create --replication-factor 1 --partitions 1 --topic test --zookeeper localhost:2181/kafka


  bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
  ## list topics
  bin/kafka-topics.sh --list --bootstrap-server localhost:9092

# Send some messages
 bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

 # Start a consumer
  bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning


http://kafka.apache.org/quickstart
