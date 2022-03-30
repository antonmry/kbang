
$ jbang -i Kafka.java

See https://kafka.apache.org/31/documentation/streams/

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

streams.close();

```
