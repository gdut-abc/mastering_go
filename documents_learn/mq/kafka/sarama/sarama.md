# 概览

sarama是一个纯Go的Apache Kafka（0.8以及之后的版本）客户端库。它包含一个高层的API（用于）和一个底层的API（用于当高层API不足够时，控制wire线路上的字节）。

为了生产消息，要么使用 AsyncProducer,要么使用SyncProducer。AsyncProducer 在一个channel上接收消息，并尽可能高效地在后台异步生产（produce）他们。这种方式在大多数情况下是首选。 SyncProducer 提供一个方法，这个方法将会阻塞直到Kafka确认这条消息是produced的。这种方式有用但是会带来如下两个注意事项：  

* 通常没有那么高效，实际的持续时间保证取决于Producer.RequiredAcks配置的值

* 在某些配置中，有时仍然会丢失SyncProducer确认的消息。

为了消费消息，使用Consumer或者Consumer-Group API.

对于底层的需求，Broker和Request/Response 对象允许对在wire上的每个连接和发送的每条消息做精确控制。Client对象提供更高层的metadata管理，这种metadata在生产者和消费者之间共享。 Request/Response对象和属性大部分都没有在文档中，因为他们和kafka文档中的协议字段完全一致。
https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol

指标都是通过 https://github.com/rcrowley/go-metrics 这个库在本地的registry中暴露的。

