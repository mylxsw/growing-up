# RabbitMQ发布订阅实战-实现延时重试队列

[TOC]

RabbitMQ是一款使用Erlang开发的开源消息队列。本文假设读者对RabbitMQ是什么已经有了基本的了解，如果你还不知道它是什么以及可以用来做什么，建议先从官网的 [RabbitMQ Tutorials](http://www.rabbitmq.com/getstarted.html) 入门教程开始学习。

本文将会讲解如何使用RabbitMQ实现延时重试和失败消息队列，实现可靠的消息消费，消费失败后，自动延时将消息重新投递，当达到一定的重试次数后，将消息投递到失败消息队列，等待人工介入处理。在这里我会带领大家一步一步的实现一个带有失败重试功能的发布订阅组件，使用该组件后可以非常简单的实现消息的发布订阅，在进行业务开发的时候，业务开发人员可以将主要精力放在业务逻辑实现上，而不需要花费时间去理解RabbitMQ的一些复杂概念。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 概要

我们将会实现如下功能

- 结合RabbitMQ的Topic模式和Work Queue模式实现生产方产生消息，消费方按需订阅，消息投递到消费方的队列之后，多个worker同时对消息进行消费
- 结合RabbitMQ的 [Message TTL](http://www.rabbitmq.com/ttl.html) 和 [Dead Letter Exchange](http://www.rabbitmq.com/dlx.html) 实现消息的延时重试功能
- 消息达到最大重试次数之后，将其投递到失败队列，等待人工介入处理bug后，重新将其加入队列消费

具体流程见下图

![RabbitMQ消息处理流程](https://oayrssjpa.qnssl.com/:Users:mylxsw:Downloads:RabbitMQ消息处理流程 -2-.jpg)


1. 生产者发布消息到主Exchange
2. 主Exchange根据Routing Key将消息分发到对应的消息队列
3. 多个消费者的worker进程同时对队列中的消息进行消费，因此它们之间采用“竞争”的方式来争取消息的消费
4. 消息消费后，不管成功失败，都要返回ACK消费确认消息给队列，避免消息消费确认机制导致重复投递，同时，如果消息处理成功，则结束流程，否则进入重试阶段
5. 如果重试次数小于设定的最大重试次数（3次），则将消息重新投递到Retry Exchange的重试队列
6. 重试队列不需要消费者直接订阅，它会等待消息的有效时间过期之后，重新将消息投递给Dead Letter Exchange，我们在这里将其设置为主Exchange，实现延时后重新投递消息，这样消费者就可以重新消费消息
7. 如果三次以上都是消费失败，则认为消息无法被处理，直接将消息投递给Failed Exchange的Failed Queue，这时候应用可以触发报警机制，以通知相关责任人处理
8. 等待人工介入处理（解决bug）之后，重新将消息投递到主Exchange，这样就可以重新消费了

## 技术实现

 [Linus Torvalds](https://en.wikiquote.org/wiki/Linus_Torvalds) 曾经说过
 
> Talk is cheap. Show me the code

我分别用Java和PHP实现了本文所讲述的方案，读者可以通过参考代码以及本文中的基本步骤来更好的理解

- [rabbitmq-pubsub-php](https://github.com/mylxsw/rabbitmq-pubsub-php)
- [rabbitmq-pubsub-java](https://github.com/mylxsw/rabbitmq-pubsub-java)

### 创建Exchange

为了实现消息的延时重试和失败存储，我们需要创建三个Exchange来处理消息。

- **master** 主Exchange，发布消息时发布到该Exchange
- **master.retry** 重试Exchange，消息处理失败时（3次以内），将消息重新投递给该Exchange
- **master.failed** 失败Exchange，超过三次重试失败后，消息投递到该Exchange

所有的Exchange声明(declare)必须使用以下参数

| 参数  | 值  | 说明
| ------------ | ------------ | --------- |
| exchange  | -  | Exchange名称 |
| type  | topic  | Exchange 类型 |
| passive | false | 如果Exchange已经存在，则返回成功，不存在则创建 |
| durable | true | 持久化存储Exchange，这里仅仅是Exchange本身持久化，消息和队列需要单独指定其持久化 |
| no-wait | false | 该方法需要应答确认 |

Java代码

```java
// 声明Exchange：主体，失败，重试
channel.exchangeDeclare("master", "topic", true);
channel.exchangeDeclare("master.retry", "topic", true);
channel.exchangeDeclare("master.failed", "topic", true);
```

PHP代码

```php
// 普通交换机
$this->channel->exchange_declare('master', 'topic', false, true, false);
// 重试交换机
$this->channel->exchange_declare('master.retry', 'topic', false, true, false);
// 失败交换机
$this->channel->exchange_declare('master.failed', 'topic', false, true, false);
```

### 消息发布

消息发布时，使用`basic_publish`方法，参数如下

| 参数  | 值  | 说明  |
| ------------ | ------------ | ------------ |
| message  | -  | 发布的消息对象  |
| exchange  | **master**  | 消息发布到的Exchange  |
| routing-key  | -  | 路由KEY，用于标识消息类型  |
| mandatory  | false  | 是否强制路由，指定了该选项后，如果没有订阅该消息，则会返回路由不可达错误  |
| immediate | false | 指定了当消息无法直接路由给消费者时如何处理 |

发布消息时，对于`message`对象，其内容建议使用**json编码后的字符串**，同时消息需要标识以下属性

	'delivery_mode'=> 2 // 1为非持久化，2为持久化

Java代码

```java
channel.basicPublish(
    "master", 
    routingKey, 
    MessageProperties.PERSISTENT_BASIC, // delivery_mode
    message.getBytes()
);
```

PHP代码

```php
$msg = new AMQPMessage($message->serialize(), [
    'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT,
]);

$this->channel->basic_publish($msg, 'master', $routingKey);
```

### 消息订阅

消息订阅的实现相对复杂一些，需要完成队列的声明以及队列和Exchange的绑定。

#### Declare Queue

对于每一个订阅消息的服务，都必须创建一个该服务对应的队列，将该队列绑定到关注的路由规则，这样之后，消息生产者将消息投递给Exchange之后，就会按照路由规则将消息分发到对应的队列供消费者消费了。

消费服务需要declare三个队列

- `[queue_name]` 队列名称，格式符合 `[服务名称]@订阅服务标识`
- `[queue_name]@retry` 重试队列
- `[queue_name]@failed` 失败队列

> `订阅服务标识`是客户端自己对订阅的分类标识符，比如用户中心服务（服务名称ucenter），包含两个订阅：user和enterprise，这里两个订阅的队列名称就为 `ucenter@user`和`ucenter@enterprise`，其对应的重试队列为 `ucenter@user@retry`和`ucenter@enterprise@retry`。

Declare队列时，参数规定规则如下

| 参数  | 值  | 说明  |
| ------------ | ------------ | ------------ |
|  queue | -  | 队列名称  |
|  passive | false  | 队列不存在则创建，存在则直接成功  |
|  durable | true  | 队列持久化  |
|  exclusive | false  | 排他，指定该选项为true则队列只对当前连接有效，连接断开后自动删除  |
|  no-wait | false  |  该方法需要应答确认 |
|  auto-delete | false  |  当不再使用时，是否自动删除  |

对于`@retry`重试队列，需要指定额外参数

	'x-dead-letter-exchange' => 'master'
	'x-message-ttl'          => 30 * 1000 // 重试时间设置为30s

> 这里的两个header字段的含义是，在队列中延迟30s后，将该消息重新投递到`x-dead-letter-exchange`对应的Exchange中

Java代码

```java
// 声明监听队列
channel.queueDeclare(
    queueName, // 队列名称
    true,      // durable
    false,     // exclusive
    false,     // autoDelete
    null       // arguments
);
channel.queueDeclare(queueName + "@failed", true, false, false, null);

Map<String, Object> arguments = new HashMap<String, Object>();
arguments.put("x-dead-letter-exchange", exchangeName());
arguments.put("x-message-ttl", 30 * 1000);
channel.queueDeclare(queueName + "@retry", true, false, false, arguments);
```

PHP代码

```php
$this->channel->queue_declare($queueName, false, true, false, false, false);
$this->channel->queue_declare($failedQueueName, false, true, false, false, false);
$this->channel->queue_declare(
    $retryQueueName, // 队列名称
    false,           // passive
    true,            // durable
    false,           // exclusive
    false,           // auto_delete
    false,           // nowait
    new AMQPTable([
        'x-dead-letter-exchange' => 'master',
        'x-message-ttl'          => 30 * 1000,
    ])
);
```

#### Bind Exchange & Queue 

创建完队列之后，需要将队列与Exchange绑定（`bind`），不同队列需要绑定到之前创建的对应的Exchange上面

| Queue  | Exchange  |
| ------------ | ------------ |
| [queue_name] | master  |
| [queue_name]@retry  | master.retry  |
| [queue_name]@failed  | master.failed  |

绑定时，需要提供订阅的路由KEY，该路由KEY与消息发布时的路由KEY对应，区别是这里可以使用通配符同时订阅多种类型的消息。

| 参数  | 值  | 说明  |
| ------------ | ------------ | ------------ |
| queue  | -  | 绑定的队列  |
| exchange  | -  | 绑定的Exchange  |
| routing-key  | -  | 订阅的消息路由规则  |
| no-wait  | false  | 该方法需要应答确认  |

Java代码

```java
// 绑定监听队列到Exchange
channel.queueBind(queueName, "master", routingKey);
channel.queueBind(queueName + "@failed", "master.failed", routingKey);
channel.queueBind(queueName + "@retry", "master.retry", routingKey);
```

PHP代码

```php
$this->channel->queue_bind($queueName, 'master', $routingKey);
$this->channel->queue_bind($retryQueueName, 'master.retry', $routingKey);
$this->channel->queue_bind($failedQueueName, 'master.failed', $routingKey);
```

#### 消息消费实现

使用 `basic_consume` 对消息进行消费的时候，需要注意下面参数

| 参数  | 值  | 说明  |
| ------------ | ------------ | ------------ |
|  queue | - | 消费的队列名称  |
|  consumer-tag | -  |  消费者标识，留空即可 |
|  no_local | false  |  如果设置了该字段，服务器将不会发布消息到 发布它的客户端 |
|  no_ack | false  | 需要消费确认应答  |
|  exclusive | false  | 排他访问，设置后只允许当前消费者访问该队列  |
|  nowait | false  | 该方法需要应答确认  |

消费端在消费消息时，需要从消息中获取消息被消费的次数，以此判断该消息处理失败时重试还是发送到失败队列。

Java代码

```java
protected Long getRetryCount(AMQP.BasicProperties properties) {
	Long retryCount = 0L;
	try {
		Map<String, Object> headers = properties.getHeaders();
		if (headers != null) {
			if (headers.containsKey("x-death")) {
				List<Map<String, Object>> deaths = (List<Map<String, Object>>) headers.get("x-death");
				if (deaths.size() > 0) {
					Map<String, Object> death = deaths.get(0);
					retryCount = (Long) death.get("count");
				}
			}
		}
	} catch (Exception e) {}

	return retryCount;
}
```

PHP代码

```php
protected function getRetryCount(AMQPMessage $msg): int
{
	$retry = 0;
	if ($msg->has('application_headers')) {
		$headers = $msg->get('application_headers')->getNativeData();
		if (isset($headers['x-death'][0]['count'])) {
			$retry = $headers['x-death'][0]['count'];
		}
	}

	return (int)$retry;
}
```

消息消费完成后，需要发送消费确认消息给服务端，使用`basic_ack`方法

	ack(delivery-tag=消息的delivery-tag标识)
	
Java代码

```java
// 消息消费处理
Consumer consumer = new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope,
                               AMQP.BasicProperties properties, byte[] body) throws IOException {
        ...
        // 注意，由于使用了basicConsume的autoAck特性，因此这里就不需要手动执行
        // channel.basicAck(envelope.getDeliveryTag(), false);
    }
};
// 执行消息消费处理
channel.basicConsume(
    queueName, 
    true, // autoAck
    consumer
);
```

PHP代码

```php
$this->channel->basic_consume(
    $queueName,
    '',    // customer_tag
    false, // no_local
    false, // no_ack
    false, // exclusive
    false, // nowait
    function (AMQPMessage $msg) use ($queueName, $routingKey, $callback) {
        ...
        $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
    }
);
```

如果消息处理中出现异常，应该将该消息重新投递到重试Exchange，等待下次重试

	basic_publish(msg, 'master.retry', routing-key)
	ack(delivery-tag) // 不要忘记了应答消费成功消息

如果判断重试次数大于3次，仍然处理失败，则应该讲消息投递到失败Exchange，等待人工处理

	basic_publish(msg, 'master.failed', routing-key)
	ack(delivery-tag) // 不要忘记了应答消费成功消息

> 一定不要忘记ack消息，因为重试、失败都是通过将消息重新投递到重试、失败Exchange来实现的，如果忘记ack，则该消息在超时或者连接断开后，会重新被重新投递给消费者，如果消费者依旧无法处理，则会造成死循环。

Java代码

```java
try {
    String message = new String(body, "UTF-8");
    // 消息处理函数
    handler.handle(message, envelope.getRoutingKey());

} catch (Exception e) {
    long retryCount = getRetryCount(properties);
    if (retryCount > 3) {
        // 重试次数大于3次，则自动加入到失败队列
        channel.basicPublish("master.failed", envelope.getRoutingKey(), MessageProperties.PERSISTENT_BASIC, body);
    } else {
        // 重试次数小于3，则加入到重试队列，30s后再重试
        channel.basicPublish("master.retry", envelope.getRoutingKey(), properties, body);
    }
}
```

#### 失败任务重试

如果任务重试三次仍未成功，则会被投递到失败队列，这时候需要人工处理程序异常，处理完毕后，需要将消息重新投递到队列进行处理，这里唯一需要做的就是从失败队列订阅消息，然后获取到消息后，清空其`application_headers`头信息，然后重新投递到`master`这个Exchange即可。

Java代码

```java
channel.basicPublish(
    'master', 
    envelope.getRoutingKey(),
    MessageProperties.PERSISTENT_BASIC,
    body
);
```

PHP代码

```php
$msg->set('application_headers', new AMQPTable([]));
$this->channel->basic_publish(
    $msg,
    'master',
    $msg->get('routing_key')
);
```

## 总结

使用RabbitMQ时，实现延时重试和失败队列的方式并不仅仅局限于本文中描述的方法，如果读者有更好的实现方案，欢迎拍砖，在这里我也只是抛砖引玉了。本文中讲述的方法还有很多优化空间，读者也可以试着去改进其实现方案，比如本文中使用了三个Exchagne，是否只使用一个Exchange也能实现本文中所讲述的功能。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。


