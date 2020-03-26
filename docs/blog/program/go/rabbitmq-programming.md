---
title: RabbimtMQ Programming
date: 2018-02-01
categories:
- 博客
---
#### AMQP Concepts

[AMQP Concepts](https://www.rabbitmq.com/tutorials/amqp-concepts.html)

#### Confustion between Connection and Channel

1. A Connection represents a real TCP connection to the message broker, whereas a Channel is a virtual connection (AMPQ connection) inside it. This way you can use as many (virtual) connections as you want inside your application without overloading the broker with TCP connections.

2. You can use one Channel for everything. However, if you have multiple threads, it's suggested to use a different Channel for each thread. There is no direct relation between Channel and Queue. A Channel is used to send AMQP commands to the broker. This can be the creation of a queue or similar, but these concepts are not tied together.

3. Each Consumer runs in its own thread allocated from the consumer thread pool. If multiple Consumers are subscribed to the same Queue, the broker uses round-robin to distribute the messages between them equally.


#### Reference
- [The RabbitMQ Threading Model](http://moi.vonos.net/bigdata/rabbitmq-threading/)

