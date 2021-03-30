# RabbitMQ

> RabbitMQ是一个由erlang开发的AMQP(Advanved Message Queue)的开源实现。

## 135.rabbitmq 的使用场景有哪些？

2.1异步处理
场景说明：用户注册后，需要发注册邮件和注册短信,传统的做法有两种1.串行的方式;2.并行的方式
(1)串行方式:将注册信息写入数据库后,发送注册邮件,再发送注册短信,以上三个任务全部完成后才返回给客户端。 这有一个问题是,邮件,短信并不是必须的,它只是一个通知,而这种做法让客户端等待没有必要等待的东西.

(2)并行方式:将注册信息写入数据库后,发送邮件的同时,发送短信,以上三个任务完成后,返回给客户端,并行的方式能提高处理的时间。

假设三个业务节点分别使用50ms,串行方式使用时间150ms,并行使用时间100ms。虽然并性已经提高的处理时间,但是,前面说过,邮件和短信对我正常的使用网站没有任何影响，客户端没有必要等着其发送完成才显示注册成功,英爱是写入数据库后就返回.
(3)消息队列
引入消息队列后，把发送邮件,短信不是必须的业务逻辑异步处理

由此可以看出,引入消息队列后，用户的响应时间就等于写入数据库的时间+写入消息队列的时间(可以忽略不计),引入消息队列后处理后,响应时间是串行的3倍,是并行的2倍。

2.2 应用解耦
场景：双11是购物狂节,用户下单后,订单系统需要通知库存系统,传统的做法就是订单系统调用库存系统的接口.

这种做法有一个缺点:

当库存系统出现故障时,订单就会失败。(这样马云将少赚好多好多钱^ ^)
订单系统和库存系统高耦合.
引入消息队列


订单系统:用户下单后,订单系统完成持久化处理,将消息写入消息队列,返回用户订单下单成功。

库存系统:订阅下单的消息,获取下单消息,进行库操作。
就算库存系统出现故障,消息队列也能保证消息的可靠投递,不会导致消息丢失(马云这下高兴了).
流量削峰
流量削峰一般在秒杀活动中应用广泛
场景:秒杀活动，一般会因为流量过大，导致应用挂掉,为了解决这个问题，一般在应用前端加入消息队列。
作用:
1.可以控制活动人数，超过此一定阀值的订单直接丢弃(我为什么秒杀一次都没有成功过呢^^)
2.可以缓解短时间的高流量压垮应用(应用程序按自己的最大处理能力获取订单)

1.用户的请求,服务器收到之后,首先写入消息队列,加入消息队列长度超过最大值,则直接抛弃用户请求或跳转到错误页面.
2.秒杀业务根据消息队列中的请求信息，再做后续处理.

## 136.rabbitmq 有哪些重要的角色？

生产者：消息的创建者，负责创建和推送数据到消息服务器

消费者：消息的接收方，用于处理数据和确认消息

代理：就是RabbitMQ本身，用于扮演快递的角色，本身并不生产消息

## 137.rabbitmq 有哪些重要的组件？

ConnectionFactory(连接管理器)：应用程序与RabbitMQ之间建立连接的管理器

Channel(信道)：消息推送使用的通道

Exchange(交换器)：用于接受、分配消息

Queue(队列)：用于存储生产者的消息

RoutingKey(路由键)：用于把生产者的数据分配到交换器上

BindKey(绑定键)：用于把交换器的消息绑定到队列上

## 138.rabbitmq 中 vhost 的作用是什么？

vhost本质上是一个mini版的RabbitMQ服务器，拥有自己的队列、绑定、交换器和权限控制；

vhost通过在各个实例间提供逻辑上分离，允许你为不同应用程序安全保密地运行数据；

vhost是AMQP概念的基础，必须在连接时进行指定，RabbitMQ包含了默认vhost：“/”；

当在RabbitMQ中创建一个用户时，用户通常会被指派给至少一个vhost，并且只能访问被指派vhost内的队列、交换器和绑定，vhost之间是绝对隔离的。

**vhost可以理解为虚拟broker，即mini-RabbitMQ server，其内部均含有独立的queue、bind、exchange等，最重要的是拥有独立的权限系统，可以做到vhost范围内的用户控制。当然，从RabbitMQ全局角度，vhost可以作为不同权限隔离的手段(一个典型的例子，不同的应用可以跑在不同的vhost中)。**

## 139.rabbitmq 的消息是怎么发送的？



## 140.rabbitmq 怎么保证消息的稳定性？

• 生产者发出后保证到达了MQ。
为了解决这个问题，RabbitMQ引入了事务机制和发送方确认机制（publisher confirm），由于事务机制过于耗费性能所以一般不用。另一个就是消息发送到MQ那端之后，MQ会回一个确认收到的消息给我们。

• MQ收到消息保证分发到了消息对应的Exchange。
消息找不到对应的Exchange。找不到对应的Queue。这两种情况都可以用RabbitMQ提供的mandatory参数来解决，它会设置消息投递失败的策略，有两种策略：自动删除或返回到客户端。

• Exchange分发消息入队之后保证消息的持久性。
对消息做持久化，以便MQ重新启动之后消息还能重新恢复过来。消息的持久化要做，还要做队列的持久化和Exchange的持久化。创建Exchange和队列时只要设置好持久化，发送的消息默认就是持久化消息。如果出现服务器宕机或者磁盘损坏则上面的手段统统无效，必须引入镜像队列，做异地多活来抵御这种不可抗因素。

• 消费者收到消息之后保证消息的正确消费。
消费者的消息确认。

案例
消息是先入库，然后生产者将数据包装成消息发给MQ。经过消费者消费之后对DB数据的状态进行更改。这中间有任何步骤失败，数据的状态都是没有更新的。这时通过一个定时任务不停的去刷库，找到有问题的数据将它重新扔到生产者那里进行重新投递。

## 141.rabbitmq 怎么避免消息丢失？

rabbitMQ消息可能丢失的情况：
1.发送方发出消息但没有进入队列。
2.接收者接到消息，但处理过程出现错误。
3.队列或者交换机宕机。

针对上面的三种情况，rabbitMQ有三种应对措施。

**publisher-confirms（发送方确认模式）**
将信道设置成confirm模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的ID。
一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会发送一个确认给生产者（包含消息唯一ID）。
如果RabbitMQ发生内部错误从而导致消息丢失，会发送一条nack（not acknowledged，未确认）消息。
发送方确认模式是异步的，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息
使用Spring AMQP时，我们可以人为配置重发时长。

**消息确认机制（ACK）**
当消费者获取消息后，会向RabbitMQ发送回执ACK，告知消息已经被接收。不过这种回执ACK分两种情况：

自动ACK：消息一旦被接收，消费者自动发送ACK

- 如果消息不太重要，丢失也没有影响，那么自动ACK会比较方便
  手动ACK：消息接收后，不会发送ACK，需要手动调用

- 如果消息非常重要，不容丢失。那么最好在消费完成后手动ACK，否则接收消息后就自动ACK，RabbitMQ就会把消息从队列中删除。如果此时消费者宕机，那么消息就丢失了。、

**持久化**
消息持久化，当然前提是队列和交换机必须持久化
RabbitMQ确保持久性消息能从服务器重启中恢复的方式是，将它们写入磁盘上的一个持久化日志文件，当发布一条持久性消息到持久交换器上时，Rabbit会在消息提交到日志文件后才发送响应。
一旦消费者从持久队列中消费了一条持久化消息，RabbitMQ会在持久化日志中把这条消息标记为等待垃圾收集。如果持久化消息在被消费之前RabbitMQ重启，那么Rabbit会自动重建交换器和队列（以及绑定），并重新发布持久化日志文件中的消息到合适的队列。

## 142.要保证消息持久化成功的条件有哪些？

**声明队列必须设置持久化 durable 设置为 true.**

**消息推送投递模式必须设置持久化，deliveryMode 设置为 2（持久）。**

**消息已经到达持久化交换器。**

**消息已经到达持久化队列。**

以上四个条件都满足才能保证消息持久化成功。

## 143.rabbitmq 持久化有什么缺点？

持久化的缺地就是降低了服务器的吞吐量，因为使用的是磁盘而非内存存储，从而降低了吞吐量。可尽量使用 ssd 硬盘来缓解吞吐量的问题

## 144.rabbitmq 有几种广播类型？

**direct**（默认方式）：最基础最简单的模式，发送方把消息发送给订阅方，如果有多个订阅者，默认采取轮询的方式进行消息发送。

**headers**：与 direct 类似，只是性能很差，此类型几乎用不到。

**fanout**：分发模式，把消费分发给所有订阅者。

**topic**：匹配订阅模式，使用正则匹配到消息队列，能匹配到的都能接收到。

## 145.rabbitmq 怎么实现延迟消息队列？

**什么是延迟队列**
延迟队列存储的对象肯定是对应的延迟消息，所谓”延迟消息”是指当消息被发送以后，并不想让消费者立即拿到消息，而是等待指定时间后，消费者才拿到这个消息进行消费。

场景一：在订单系统中，一个用户下单之后通常有30分钟的时间进行支付，如果30分钟之内没有支付成功，那么这个订单将进行一场处理。这是就可以使用延迟队列将订单信息发送到延迟队列。

场景二：用户希望通过手机远程遥控家里的智能设备在指定的时间进行工作。这时候就可以将用户指令发送到延迟队列，当指令设定的时间到了再将指令推送到只能设备。

***

**RabbitMQ怎么实现延迟队列**
AMQP协议，以及RabbitMQ本身没有直接支持延迟队列的功能，但是可以通过TTL和DLX模拟出延迟队列的功能。

**TTL（Time To Live）**
RabbitMQ可以针对Queue和Message设置 x-message-tt，来控制消息的生存时间，如果超时，则消息变为dead letter
RabbitMQ针对队列中的消息过期时间有两种方法可以设置。

- A: 通过队列属性设置，队列中所有消息都有相同的过期时间。
- B: 对消息进行单独设置，每条消息TTL可以不同。
  如果同时使用，则消息的过期时间以两者之间TTL较小的那个数值为准。消息在队列的生存时间一旦超过设置的TTL值，就成为dead letter

详细可以参考：RabbitMQ之TTL（Time-To-Live 过期时间）

**DLX (Dead-Letter-Exchange)**

RabbitMQ的Queue可以配置x-dead-letter-exchange 和x-dead-letter-routing-key（可选）两个参数，如果队列内出现了dead letter，则按照这两个参数重新路由。

- x-dead-letter-exchange：出现dead letter之后将dead letter重新发送到指定exchange

- x-dead-letter-routing-key：指定routing-key发送

队列出现dead letter的情况有：

- 消息或者队列的TTL过期
- 队列达到最大长度
- 消息被消费端拒绝（basic.reject or basic.nack）并且requeue=false

利用DLX，当消息在一个队列中变成死信后，它能被重新publish到另一个Exchange。这时候消息就可以重新被消费。

详细可以参考： RabbitMQ之死信队列

代码示例
首先建立2个exchange和2个queue：

- exchange_delay_begin：这个是producer端发送时调用的exchange, 将消息发送至queue_dealy_begin中。
- queue_delay_begin: 通过routingKey="delay"绑定exchang_delay_begin, 同时配置DLX=exchange_delay_done, 当消息变成死信时，发往exchange_delay_done中。
- exchange_delay_done: 死信的exchange, 如果不配置x-dead-letter-routing-key则采用原有默认的routingKey，即queue_delay_begin绑定exchang_delay_beghin采用的“delay”。
- queue_delay_done：消息在TTL到期之后，最终通过exchang_delay_done发送值此queue，消费端通过消费此queue的消息，即可以达到延迟的效果。

1. 建立exchange和queue的代码（当然这里可以通过RabbitMQ的管理界面来实现，无需code相关代码）：

```java
channel.exchangeDeclare("exchange_delay_begin", "direct", true);
channel.exchangeDeclare("exchange_delay_done", "direct", true);

Map<String, Object> args = new HashMap<String, Object>();
args.put("x-dead-letter-exchange", "exchange_delay_done");
channel.queueDeclare("queue_delay_begin", true, false, false, args);
channel.queueDeclare("queue_delay_done", true, false, false, null);

channel.queueBind("queue_delay_begin", "exchange_delay_begin", "delay");
channel.queueBind("queue_delay_done", "exchange_delay_done", "delay");
```

2. consumer端代码：

```java
QueueingConsumer consumer = new QueueingConsumer(channel);
channel.basicConsume("queue_delay_done", false, consumer);

while (true) {
    QueueingConsumer.Delivery delivery = consumer.nextDelivery();
    String msg = new String(delivery.getBody());
    System.out.println("receive msg time:" + new Date() + ", msg body:" + msg);
    channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
}
```

3. producer端代码：设置消息的延迟时间为1min。

```JAVA
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
builder.expiration("60000");//设置消息TTL
builder.deliveryMode(2);//设置消息持久化
AMQP.BasicProperties properties = builder.build();

String message = String.valueOf(new Date());
channel.basicPublish("exchange_delay_begin","delay",properties,message.getBytes());
```

在创建完exchange和queue之后，首先执行consumer端的代码，之后执行producer端的代码，待producer发送完毕之后，查看consumer端的输出：

```java
receive msg time:Tue Feb 14 21:06:19 CST 2017, msg body:Tue Feb 14 21:05:19 CST 2017
```

可以看到延迟1min消费了相关消息。

## 146.rabbitmq 集群有什么用？

集群主要有以下两个用途：

**高可用**：某个服务器出现问题，整个 RabbitMQ 还可以继续使用；

**高容量**：集群可以承载更多的消息量。

## 147.rabbitmq 节点的类型有哪些？

磁盘节点：消息会存储到磁盘。

内存节点：消息都存储在内存中，重启服务器消息丢失，性能高于磁盘类型。

## 148.rabbitmq 集群搭建需要注意哪些问题？

各节点之间使用“–link”连接，此属性不能忽略。

各节点使用的 erlang cookie 值必须相同，此值相当于“秘钥”的功能，用于各节点的认证。

整个集群中必须包含一个磁盘节点。

## 149.rabbitmq 每个节点是其他节点的完整拷贝吗？为什么？

不是，原因有以下两个：

1. 存储空间的考虑：如果每个节点都拥有所有队列的完全拷贝，这样新增节点不但没有新增存储空间，反而增加了更多的冗余数据；
2. 性能的考虑：如果每条消息都需要完整拷贝到每一个集群节点，那新增节点并没有提升处理消息的能力，最多是保持和单节点相同的性能甚至是更糟。

**RabbitMQ原理**

RabbitMQ分布式部署有3种方式：集群、Federation和Shovel。这三种方式并不是互斥的，可以根据需求选择相互组合来达到目的，后两者都是以插件的形式进行设计，复杂性相对高，此篇只聊一下RabbitMQ自带的内建集群。

我们把部署RabbitMQ的机器称为节点，也就是broker。broker有2种类型节点：**磁盘节点和内存节点**。顾名思义，磁盘节点的broker把元数据存储在磁盘中，内存节点把元数据存储在内存中，很明显，磁盘节点的broker在重启后元数据可以通过读取磁盘进行重建，保证了元数据不丢失，内存节点的broker可以获得更高的性能，但在重启后元数据就都丢了。元数据包含以下内容：

- queue元数据：queue名称、属性
- exchange：exchange名称、属性（注意此处是exchange本身）
- binding元数据：exchange和queue之间、exchange和exchange之间的绑定关系
- vhost元数据：vhost内部的命名空间、安全属性数据等

![rabbitmq.jpg](E:\.学习\learning-notes\面试题\221608646563655061223.jpg)



队列所在的节点称为宿主节点。

**队列创建时**，只会在宿主节点创建队列的进程，**宿主节点包含完整的队列信息**，包括元数据、状态、内容等等。因此，只有队列的宿主节点才能知道队列的所有信息。

**队列创建后**，集群**只会同步队列和交换器的元数据到集群中的其他节点**，并**不会同步队列本身**，因此非宿主节点就只知道队列的元数据和指向该队列宿主节点的指针。

这样的设计，保证了不论从哪个broker中均可以消费所有队列的数据，并分担了负载，因此，增加broker可以线性提高服务的性能和吞吐量。

但该方案也有显著的缺陷，那就是不能保证消息不会丢失。当集群中**某一节点崩溃时**，崩溃节点所在的队列进程和关联的绑定都会消失，附加在那些队列上的消费者也会丢失其订阅信息，匹配该队列的新消息也会丢失。

**崩溃节点重启后**，需要从**磁盘节点**中同步元数据信息，并重建队列，所以，集群中要求**必须至少有一个broker为磁盘节点**，以保证集群的可用性。

为何集群不将队列内容和状态复制到所有节点上呢？有2个原因：

如果包含了完整队列，那么所有节点将会是同样的数据拷贝，也就是所有节点均互为镜像，无法拓宽负载，延展性不高

每次的同步都会让消息同步到其他节点上并落盘，引发大量的网络IO和磁盘IO，无法提升性能。（此处可以引申思考一下kafka中replica的分配方式）

如果**磁盘节点崩溃了，集群依然可以继续路由消息**（因为其他节点元数据在还存在），但无法做以下操作：

- 创建队列、交换器、绑定
- 添加用户
- 更改权限
- 添加、删除集群节点

也就是说，唯一的磁盘节点崩溃后，为了保证可用性，禁用了和元数据相关的添加、修改和删除操作。**所以建立集群的时候，建议保证有2个以上的磁盘节点。**

集群添加或者删除节点时，会把变更通知到至少一个磁盘节点。在内存节点重启后，会先从磁盘节点中同步元数据，内存节点中唯一存储到磁盘的是磁盘节点的地址。

RabbitMQ采用**镜像队列**的方式保证队列的可靠性。镜像队列就是将队列镜像到集群中的其他节点，如果集群中一个节点失效了，队列就能自动的切换到镜像中的节点，一个镜像队列中包含有1个主节点master和若干个从节点slave。其主从节点包含如下几个特点：

- 消息的读写都是在master上进行，并不是读写分离
- master接收命令后会向salve进行组播，salve会命令执行顺序执行
- master失效，根据节点加入的时间，最老的slave会被提升为master
- 互为镜像的是队列，并非节点，集群中可以不同节点可以互为镜像队列，也就是说队列的master可以分布在不同的节点上

## 150.rabbitmq 集群中唯一一个磁盘节点崩溃了会发生什么情况？

如果唯一磁盘的磁盘节点崩溃了，

不能进行以下操作：

1. 不能创建队列
2. 不能创建交换器
3. 不能创建绑定
4. 不能添加用户
5. 不能更改权限
6. 不能添加和删除集群节点

唯一磁盘节点崩溃了，集群是可以保持运行的，但你不能更改任何东西

## 151.rabbitmq 对集群节点停止顺序有要求吗？

RabbitMQ 对集群的停止的顺序是有要求的，应该先关闭内存节点，最后再关闭磁盘节点。如果顺序恰好相反的话，可能会造成消息的丢失。