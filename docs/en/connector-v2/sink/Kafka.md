# Kafka

> Kafka sink connector
## Description

Write Rows to a Kafka topic.

## Key features

- [x] [exactly-once](../../concept/connector-v2-features.md)

By default, we will use 2pc to guarantee the message is sent to kafka exactly once.

- [ ] [schema projection](../../concept/connector-v2-features.md)

## Options

| name                 | type                  | required | default value |
|----------------------|-----------------------| -------- | ------------- |
| topic                | string                | yes      | -             |
| bootstrap.servers    | string                | yes      | -             |
| kafka.*              | kafka producer config | no       | -             |
| semantic             | string                | no       | NON           |
| partition_key_fields | array                 | no       | -             |
| partition            | int                   | no       | -             |
| assign_partitions    | array                 | no       | -             |
| transaction_prefix   | string                | no       | -             |
| format               | String                | no       | json          |
| field_delimiter      | String                | no       | ,             |
| common-options       | config                | no       | -             |

### topic [string]

Kafka Topic.

### bootstrap.servers [string]

Kafka Brokers List.

### kafka.* [kafka producer config]

In addition to the above parameters that must be specified by the `Kafka producer` client, the user can also specify multiple non-mandatory parameters for the `producer` client, covering [all the producer parameters specified in the official Kafka document](https://kafka.apache.org/documentation.html#producerconfigs).

The way to specify the parameter is to add the prefix `kafka.` to the original parameter name. For example, the way to specify `request.timeout.ms` is: `kafka.request.timeout.ms = 60000` . If these non-essential parameters are not specified, they will use the default values given in the official Kafka documentation.

### semantic [string]

Semantics that can be chosen EXACTLY_ONCE/AT_LEAST_ONCE/NON, default NON.

In EXACTLY_ONCE, producer will write all messages in a Kafka transaction that will be committed to Kafka on a checkpoint.

In AT_LEAST_ONCE, producer will wait for all outstanding messages in the Kafka buffers to be acknowledged by the Kafka producer on a checkpoint.

NON does not provide any guarantees: messages may be lost in case of issues on the Kafka broker and messages may be duplicated.

### partition_key_fields [array]

Configure which fields are used as the key of the kafka message.

For example, if you want to use value of fields from upstream data as key, you can assign field names to this property.

Upstream data is the following:

| name | age  | data          |
| ---- | ---- | ------------- |
| Jack | 16   | data-example1 |
| Mary | 23   | data-example2 |

If name is set as the key, then the hash value of the name column will determine which partition the message is sent to.

If not set partition key fields, the null message key will be sent to.

### partition [int]

We can specify the partition, all messages will be sent to this partition.

### assign_partitions [array]

We can decide which partition to send based on the content of the message. The function of this parameter is to distribute information.

For example, there are five partitions in total, and the assign_partitions field in config is as follows:
assign_partitions = ["shoe", "clothing"]

Then the message containing "shoe" will be sent to partition zero ,because "shoe" is subscripted as zero in assign_partitions, and the message containing "clothing" will be sent to partition one.For other messages, the hash algorithm will be used to divide them into the remaining partitions.

This function by `MessageContentPartitioner` class implements `org.apache.kafka.clients.producer.Partitioner` interface.If we need custom partitions, we need to implement this interface as well.

### transaction_prefix [string]

If semantic is specified as EXACTLY_ONCE, the producer will write all messages in a Kafka transaction.
Kafka distinguishes different transactions by different transactionId. This parameter is prefix of  kafka  transactionId, make sure different job use different prefix.

### format

Data format. The default format is json. Optional text format. The default field separator is ",".
If you customize the delimiter, add the "field_delimiter" option.

### field_delimiter

Customize the field delimiter for data format.

### common options [config]

Sink plugin common parameters, please refer to [Sink Common Options](common-options.md) for details.

## Examples

```hocon
sink {

  kafka {
      topic = "seatunnel"
      bootstrap.servers = "localhost:9092"
      partition = 3
      format = json
      kafka.request.timeout.ms = 60000
      semantics = EXACTLY_ONCE
  }
  
}
```

## Changelog

### 2.3.0-beta 2022-10-20

- Add Kafka Sink Connector

### next version

- [Improve] Support to specify multiple partition keys [3230](https://github.com/apache/incubator-seatunnel/pull/3230)
- [Improve] Add text format for kafka sink connector [3711](https://github.com/apache/incubator-seatunnel/pull/3711)