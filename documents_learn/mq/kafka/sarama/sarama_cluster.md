# Sarama集群

Sarama的集群扩展。Apache Kafka 0.9及其以后的版本的Go client库。

## 实例

### 多路复用模式

消费者有两种操作模式。默认的多路复用模式下，多个topic和partition的消息和错误都传递给单个channel。

```golang
package main

import (
    "fmt"
    "log"
    "os"
	"os/signal"

	cluster "github.com/bsm/sarama-cluster"
)

func main() {
    // init config, enable errors and notifications
    config := cluster.NewConfig()
    config.Consumer.Return.Errors = true
    config.Group.Return.Notifications = true

    // init consumer
    brokers := []string{"127.0.0.1:9092"}
    topics := []string{"my_topic", "other_topic"}
    consumer, err := cluster.NewConsumer(brokers, "my-consumer-group", topics, config)
    if err != nil {
        panic(err)
    }
    defer consumer.Close()

    // trap SIGINT to trigger a shutdonw
    singnals := make(chan os.Signal, 1)
    signal.Notify(signals, os.Interrupt)

    // 多个协程，每个协程分别处理consumer的错误、消息、通知
    // consumer errors
    go func() {
        for err := range consumer.Errors() {
            log.Printf("Error: %s\n", err.Error)
        }
    }
    // 消费通知
    go func() {
        for ntf := range consumer.Notifications() {
            log.Printf("Rebalanced: %+v\n", ntf)
        }
    }
    // 消费消息，监控信号
    for {
        select {
            // 直接从整个consumer来消费
            case msg, ok := <-consumer.Message() {
                if ok {
                    fmt.Fprintf(os.Stdout, "%s/%d/%d/\t%s\5%s\n",
                        msg.Topic, msg.Partition, msg.Offset,
                        msg.Key, msg.Value)
                    // 标志消息已经处理
                    consumer.MarkOffset(msg, "")
                }
            }
            case <- signals:
                return
        }
    }
}
```

### 分区模式

对于需要访问指定的partition的用户，可以使用分区模式。这种分区模式可以向partition级别的consumer暴露访问方式。

```golang
package mian
import (
  "fmt"
  "log"
  "os"
  "os/signal"

  cluster "github.com/bsm/sarama-cluster"
)
func main() {
    // init config
    config := cluster.NewConfig()
    config.Group.Mode = cluster.ConsumerModePartitions

    // init consumer
    brokers := []string{"127.0.0.1:9092"}
    topics := []string{"my_topic", "other_topic"}
    consumer, err := cluster.NewConsumer(brokers, "my-consumer-group", topics, config)
    if err != nil {
        panic(err)
    }
    defer consumer.Close()

    // trap SIGINT to trigger a shutdown
    signals := make(chan os.Signal, 1)
    signal.Notify(signals, os.Interrupt)

    // consume partitions
    for {
        select {
        case part, ok := <-consumer.Partitions():
            if !ok {
                return
            }
            // 启动一个独立的goroutine来消费消息
            go func(pc cluster.PartitionConsumer) {
                for msg := range pc.Messages() {
                    fmt.Fprintf(os.Stdout, "%s/%d/%d\t%s\t%s\n",
                        msg.Topic, msg.Partition, msg.Offset, msg.Key, msg.Value)
                    consumer.MarkOffset(msg, "")   // mark message as processed
                }
            }(part)
        case <-signals:
            return
        }
    }
}
```

## 概览

cluster包给Sarama提供集群扩展。让用户从多个、已经平衡的节点中消费topic。

需要Kafka v0.9以上的版本，并且下面的步骤指南：

https://cwiki.apache.org/confluence/display/KAFKA/Kafka+0.9+Consumer+Rewrite+Design


关于api其实重点就是几个对象而已：

* Client类型

* Config类型

* Consumer类型

* PartitionConsumer类型

## Client类型

```golang
type Client struct {
    sarama.Client
    // 其他已经过滤的或者没有导出的字段
}
Client是一个group client
func NewClient(addrs []string, config *Config) (*Client, error) 
NewClient 创建一个新的client实例。
func (c *Client) ClusterConfig() *Config
ClusterConfig返回cluster配置。
```

## Config类型

```golang
type Config struct {
    sarama.Config 
    // Group 就是用来组管理属性的名字空间
    Group struct {

        // 用于分配partitions到consumers过程中使用的策略（默认使用StrategyRange）
        PartitionStrategy Strategy

        Mode ConsumerMode

        Offsets struct {
            Retry struct {
                Max int
            }
            Synchronization struct {
                DwellTime time.Duration
            }
        }
        
        Session struct {
            Timeout time.Duration
        }
        
        Heartbeat struct {
            Interval time.Duration
        }

        Return struct {
            Notifications bool
        }

        Topics struct {
            Whitelist *regexp.Regexp
            Blacklist *regexp.Regexp
        }

        Member struct {
            UserData []byte
        }

    }
}
```

## Consumer类型

```golang
type Consumer struct {
    // 包含过滤掉的或者未导出的字段
}
Consumer是一个 cluster group consumer
接下来这个例子演示如何使用consumer从多个topic
```