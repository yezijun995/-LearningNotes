# RabbitMQ



## 什么是MQ

``MQ`` (Message Queue)，翻译为``消息队列``，通过典型的 `生产者`和`消费者`模型,生产者不断向消息队列中生产消息，消费者不断的从队列中获取消息。因为消息的生产和消费都是异步的，而且只关心消息的发送和接收，没有业务逻辑的侵入,轻松的实现系统间解耦。

RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，多用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

## Dcoker下RabbitMQ的安装

```bash
# 拉取rabbitMQ镜像
docker pull rabbitmq

# 运行镜像文件并创建容器
#	-d 后台运行，--name 容器名,-p将容器内端口映射到本机
#	-e 指定环境变量；(RABBITMQ_DEFAULT_USER：默认的用户名,RABBITMQ_DEFAULT_PASS：默认用户名的密码)
# rabbitmq:management rabbitMQ的管理界面，开启后可以使用浏览器访问management
# docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management
docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq[-e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=123456]  rabbitmq:management
```

访问``虚拟机IP地址:15672``

![](http://picturebed.yifelix.cn/rabbitmq/%2A%2A1.jpg)

可以使用默认账户``guest``登录



## RabbitMQ支持的消息类型

可以去它的官网查看

### 一、“Hello World”（简单 单生产单消费）

![](http://picturebed.yifelix.cn/rabbitmq/%2A%2A2.png)

- P：生产者，也就是要发送消息的程序
- C：消费者，消息的接受者，等待消息处理消息。
- queue：消息队列，图中红色部分，类似邮箱，是个消息缓冲器，生产者可以发送消息到队列，消费者也可以取出队列中的消息。

#### 1.java连接工具类

````java
private static ConnectionFactory connectionFactory;

static {
    //重量级资源 单例模式 类加载只执行一次
    //创建连接工厂
    connectionFactory = new ConnectionFactory();
    //设置连接Rabbit
    //主机地址
    connectionFactory.setHost("192.168.106.131");
    //RabbitMQ端口
    connectionFactory.setPort(5672);
    //虚拟主机名称
    connectionFactory.setVirtualHost("/ems");
    //连接用户名
    connectionFactory.setUsername("ems");
    //连接用户密码
    connectionFactory.setPassword("123456");
}

//定义提供连接对象的方法
public static Connection getConnection() {
    try {
        return connectionFactory.newConnection();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
````

#### 2.生产者

```java
public void testSendMessage() throws IOException, TimeoutException {
    Connection connection = RabbitMQUtils.getConnection();
    //获取连接中的通道
    Channel channel = connection.createChannel();
    //通道绑定对应消息队列
    //参数1：队列名称 如果队列不存在自动创建
    //参数2：用来定义队列是否持久化 
        	//当为false，MQ进行重启将清空不持久化的队列
    		//当为true,进行重启会保存队列与持久化的消息
    //参数3：exclusive 是否独占队列 true 独占队列 false 不独占
    	//true 只允许当前通道所绑定
    //参数4：autoDelete 是否在消费完成后(断开连接)自动删除队列 true 自动删除 false 不自动删除
    //参数5：额外附加参数
    channel.queueDeclare("hello",false,false,false,null);

    //发布消息
    //参数1：交换机名称 
    //参数2：队列名称 
    //参数3：传递消息额外设置 
    	//MessageProperties.PERSISTENT_TEXT_PLAIN 消息持久化
    //参数4：消息的具体内容
    channel.basicPublish("","hello",null,"hello rabbitmq".getBytes());
    //关闭连接
    RabbitMQUtils.closeConnectionAndChannel(channel,connection);
}
```

#### 3.消费者

```java
public static void main(String[] args) throws IOException, TimeoutException {
    //创建连接对象
    Connection connection = RabbitMQUtils.getConnection();
    //创建通道
    Channel channel = connection.createChannel();
    //通道绑定对象
    channel.queueDeclare("hello",false,false,false,null);
    //消费消息
    //参数1：消费哪个队列的消息 队列名称
    //参数2：开启消息的自动确认机制
    	//false： 不会自动确认消息
    //参数3：消费时的回调接口
    channel.basicConsume("hello",true,new DefaultConsumer(channel){

        /**
         *
         * @param consumerTag 标签
         * @param envelope 信封
         * @param properties 属性
         * @param body 消息队列中取出的消息
         * @throws IOException
         */
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            System.out.println("body:" + new String(body));
        }
    });

    //因为消费者需一直消费，所以无需关闭
}
```

### 二、 Work queues（工作队列）

![](http://picturebed.yifelix.cn/rabbitmq/%2A%2A3.png)

一个生产者多个消费者，让多个消费者绑定到一个队列，共同消费队列中的消息，每个消费者获得的消息都是唯一的。

角色：

- P：生产者，消息的发布者
- C<sub>1</sub>：消费者1，消息的接受者，等待消息处理消息
- C<sub>2</sub>：消费者2，消息的接受者，等待消息处理消息

#### 1.生产者

```java
public void testSendMessage() throws IOException, TimeoutException {
    Connection connection = RabbitMQUtils.getConnection();
	//获取连接中的通道
    Channel channel = connection.createChannel();
    //通过通道声明队列
    channel.queueDeclare("work", true, false, false, null);

    for (int i = 0; i < 10; i++) {
        //生产消息
        channel.basicPublish("", "work", null, ("work--" + i).getBytes());
    }
    RabbitMQUtils.closeConnectionAndChannel(channel, connection);
}
```

#### 2.消费者1

```java
public static void main(String[] args) throws IOException, TimeoutException {
    //获取连接对象
    Connection connection = RabbitMQUtils.getConnection();
    //创建通道
	Channel channel = connection.createChannel();
    //通道绑定对象
    channel.queueDeclare("work",false,false,false,null);
    channel.basicConsume("work",true,new DefaultConsumer(channel){

        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            System.out.println("消费者1:" + new String(body));
        }
    });
}
```



#### 3.消费者2

```java
public static void main(String[] args) throws IOException, TimeoutException {
    //创建连接对象
    Connection connection = RabbitMQUtils.getConnection();
    //创建通道
    Channel channel = connection.createChannel();
    //通道绑定对象
    channel.queueDeclare("work",false,false,false,null);
    channel.basicConsume("work",true,new DefaultConsumer(channel){

        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            System.out.println("消费者2:" + new String(body));
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    });
}
```

运行上述代码可以发现消费者1与消费者2收到的消息的是<span style="color:orange">相同</span>的，这是因为此消息类型是**轮询**机制，但是此类型在消费者执行时间不同情况下还是会造成消息队列的阻塞。

####  4.消息自动确认机制

消费者确认分两种：**自动确认**和**手动确认**

在自动确认中，消息发送到消费者之后就被认为成功消费，而在手动确认时需设置``basicQos``方法，设置能执行的消息数量，并将手动消息机制打开，在进行手动确认。

```java
public static void main(String[] args) throws IOException, TimeoutException {
    //创建连接对象
    Connection connection = RabbitMQUtils.getConnection();
    //创建通道
    Channel channel = connection.createChannel();
    //一次消费几个消息
    channel.basicQos(1);
    //通道绑定对象
    channel.queueDeclare("work",true,false,false,null);
    //参数2：开启消息的自动确认机制
    	//false： 不会自动确认消息
    channel.basicConsume("work",false,new DefaultConsumer(channel){

        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
           	//手动确认
                //参数1：手动确认消息表示
                //参数2：开启多个消息确认
            channel.basicAck(envelope.getDeliveryTag(),false);
        }
    });
}
```

### 三、Publish/Subscribe(发布订阅 fanout)

![](http://picturebed.yifelix.cn/rabbitmq/%2A%2A4.png)

此模型也称为 **Fanout** (广播模式)，在此模型下每个消费者都有自己的``queue``，每个队列都要绑定到相同的一个``Exchange``上，生产者发送的消息只能发到``Fanout Exchange``，在进行决定发送到哪个队列。队列中的消费者都能拿到消息，实现**一条消息被多个消费者消费**。

#### 1.生产者

```java
//获取连接对象
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();
//将通道声明指定交换机
    //参数1：交换机名称
    //参数2：交换机类型
        //fanout 广播类型
channel.exchangeDeclare("users","fanout");
//发送消息
channel.basicPublish("users","",null,"fanout message".getBytes());

RabbitMQUtils.closeConnectionAndChannel(channel,connection);
```

#### 2.消费者(多个消费者)

```java
//获取连接对象
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();
//通道绑定交换机
channel.exchangeDeclare("users","fanout");

//生成临时队列
String queueName = channel.queueDeclare().getQueue();
//绑定交换机和队列
channel.queueBind(queueName,"users","");

//消费消息
channel.basicConsume(queueName,true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者1： " + new String(body));
    }
});
```

运行结果：

![](http://picturebed.yifelix.cn/rabbitmq/%2A%2A6.png)

### 四、Routing（路由）

![](http://picturebed.yifelix.cn/rabbitmq/%2A%2A5.png)

> 在 ``Fanout`` 情况下，一条消息会被所有订阅的消费者所消费，但是不是每种情况下都需要已广播形式所消费，而是不同的消息被不同的队列消费这是所需要的 ``Direct`` 。

在 ``Direct`` 中队列与交换机需使用 ``RoutingKey`` 来进行绑定，生产者在向 ``Exchange`` 发送消息时，也必须指定消息的 ``RoutingKey`` ，``Exchange`` 会根据队列的 ``RoutingKey`` 与消息的 ``RoutingKey`` 来进行判断，当完全一致时才会往匹配成功的队列发送消息。

##### 1.生产者

```java
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();
//交换机名
String exchangeName = "logs_direct";
//通过通道声明交换机
    //参数1：交换机名称
    //参数2：路由模式
channel.exchangeDeclare(exchangeName,"direct");
//发送消息
String routingKey1 = "green";
channel.basicPublish(exchangeName,routingKey1,null,("使用Direct发送,routingKey:"+ routingKey1).getBytes());
String routingKey2 = "black";
channel.basicPublish(exchangeName,routingKey2,null,("使用Direct发送,routingKey:"+ routingKey2).getBytes());
String routingKey3 = "orange";
channel.basicPublish(exchangeName,routingKey3,null,("使用Direct发送,routingKey:"+ routingKey3).getBytes());

//关闭资源
RabbitMQUtils.closeConnectionAndChannel(channel,connection);
```

##### 2.消费者1(orange)

```java
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();
//交换机名
String exchangeName = "logs_direct";
//通道声明交换机以及交换的类型
channel.exchangeDeclare(exchangeName, "direct");
//创建一个临时队列
String queueName = channel.queueDeclare().getQueue();
//基于RoutingKey绑定队列和交换机
channel.queueBind(queueName, exchangeName, "orange");
//获取消费的消息
channel.basicConsume(queueName, true, new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者1：" + new String(body));
    }
});
```

##### 3.消费者2(black、green)

```java
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();
//交换机名
String exchangeName = "logs_direct";
//声明交换机与类型
channel.exchangeDeclare(exchangeName,"direct");
String queueName = channel.queueDeclare().getQueue();
//绑定RoutingKey为"black"与"green"
channel.queueBind(queueName,exchangeName,"black");
channel.queueBind(queueName,exchangeName,"green");
//消费消息
channel.basicConsume(queueName,true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者2："+ new String(body));
    }
});
```

### 五、Topics（主题模式）

![](http://picturebed.yifelix.cn/rabbitmq/%2A%2A7.png)

``Topics`` 是 ``Routing`` 的另外一种形式，它与 ``Exchange`` 与 ``Direct`` 一样，都是可以根据 ``Routing Key`` 将消息送到不同的队列中，但是 ``Topics`` 可以使队列绑定 ``Routing Key`` 的时候使用通配符，它可以由一个或多个单词组成，单词与单词之间使用 “.” 隔开。

```markdown
# 通配符
	* 只匹配一个词
	# 匹配一个或多个词
```

#### 1.生产者

```java
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();
//声明交换机与交换机类型 topic
channel.exchangeDeclare("topics","topic");
//发布消息
String routeKey = "user.save.add";
channel.basicPublish("topics",routeKey,null,("使用Topics发送,routingKey:"+ routeKey).getBytes());

//关闭连接
RabbitMQUtils.closeConnectionAndChannel(channel,connection);
```

#### 2.消费者1

```java
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();
//声明交换机以及交换机类型
channel.exchangeDeclare("topics","topic");
//创建临时队列
String queueName = channel.queueDeclare().getQueue();
//绑定队列和交换机，通配符形式
channel.queueBind(queueName,"topics","user.*");
//消费消息
channel.basicConsume(queueName,true, new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者1：" +new String(body));
    }
});
```

#### 3.消费者2

```java
Connection connection = RabbitMQUtils.getConnection();
Channel channel = connection.createChannel();
//声明交换机以及交换机类型
channel.exchangeDeclare("topics","topic");
//创建临时队列
String queueName = channel.queueDeclare().getQueue();
//绑定队列和交换机，通配符形式
channel.queueBind(queueName,"topics","user.#");
//消费消息
channel.basicConsume(queueName,true, new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者2：" +new String(body));
    }
});
```

测试结果：

如果发送``user.save`` 会被消费者1与消费者2同时接收

![](http://picturebed.yifelix.cn/rabbitmq/%2A%2A8.png)

而发送 ``user.save.add`` 时只会被消费者2所接收

![](http://picturebed.yifelix.cn/rabbitmq/%2A%2A9.png)

