## Run it

Locally:

```bash
jbang -i Kafka.java
```

Remotely:

```bash
jbang kafka@antonmry
```

## Create topics

```java
import org.apache.kafka.clients.admin.Admin;
import org.apache.kafka.clients.admin.NewTopic;
import org.apache.kafka.common.config.TopicConfig;
import kafka.*;

var props = Kafka.getAdminConfig();
var admin = Admin.create(props);

var inputTopic = new NewTopic("streams-plaintext-input", 1, (short) 1);
var result = admin.createTopics(Collections.singleton(inputTopic));
result.values().get("streams-plaintext-input").get();

var outputTopicConfig = Map.of(TopicConfig.CLEANUP_POLICY_CONFIG, TopicConfig.CLEANUP_POLICY_COMPACT);
var outputTopic = new NewTopic("streams-wordcount-output", 1, (short) 1).configs(outputTopicConfig);
var result = admin.createTopics(Collections.singleton(outputTopic));
result.values().get("streams-wordcount-output").get();

var r = admin.listTopics();
System.out.println(r.names().get());
```

## Create the stream

```java

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KTable;
import org.apache.kafka.streams.kstream.Produced;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.Arrays;
import java.util.Locale;
import java.util.Properties;


String INPUT_TOPIC = "streams-plaintext-input";
String OUTPUT_TOPIC = "streams-wordcount-output";

import kafka.*
var props = Kafka.getStreamsConfig()

StreamsBuilder builder = new StreamsBuilder();

KStream<String, String> source = builder.stream(INPUT_TOPIC);
KTable<String, Long> counts = source.flatMapValues(value -> Arrays.asList(value.toLowerCase(Locale.getDefault()).split("\\W+"))).groupBy((key, value) -> value).count();
counts.toStream().to(OUTPUT_TOPIC, Produced.with(Serdes.String(), Serdes.Long()));

var topology = builder.build()
KafkaStreams streams = new KafkaStreams(topology, props);

System.out.println(topology.describe());

streams.start();
streams.state();

//streams.close();
```

## Consumer

```java
import kafka.*
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import java.time.Duration;

var props = Kafka.getConsumerConfig()

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("streams-wordcount-output"));

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records)
        System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
}
```

## Producer

Run in a different `jshell`:

```java
import kafka.*
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

var props = Kafka.getProducerConfig()
Producer<String, String> producer = new KafkaProducer<>(props);

var response = producer.send(new ProducerRecord<>("streams-plaintext-input", "Hello Kafka from jshell"));

//producer.close();
```

## Execute the demo remotely

```sh
jbang alias add --name kafka https://github.com/antonmry/kbang/blob/HEAD/Kafka.java
vi /Users/arodriguez/.jbang/jbang-catalog.json
# Copy to https://github.com/antonmry/jbang-catalog
jbang kafka@antonmry
```

## Other tools / resources

https://kafka.apache.org/31/documentation/streams/
https://github.com/jeqo/poc-apache-kafka/tree/main/cli
https://github.com/eugenp/tutorials/tree/master/apache-kafka

## Next steps:

- [ ] Test E2E
- [ ] Build a library and deploy it to jitpack? 
- [x] Create the consumer from jshell
- [x] Create topics from jshell
- [x] Create producer from jshell
