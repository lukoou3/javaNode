# ActiveMQ准备

## JMS简介

全称：**Java Message Service** 中文：**Java消息服务**。

JMS是Java的一套API标准，最初的目的是为了使应用程序能够访问现有的MOM系统（MOM是Message Oriented Middleware的英文缩写，指的是利用高效可靠的消息传递机制进行平台无关的数据交流，并基于数据通信来进行分布式系统的集成。）；后来被许多现有的MOM供应商采用，并实现为MOM系统。【常见MOM系统包括Apache的ActiveMQ、阿里巴巴的RocketMQ、IBM的MQSeries、Microsoft的MSMQ、BEA的RabbitMQ等。（并非全部的MOM系统都遵循JMS规范）】

基于JMS实现的MOM，又被称为JMS Provider。

“消息”是在两台计算机间传送的数据单位。消息可以非常简单，例如只包含文本字符串；也可以更复杂，可能包含嵌入对象。

消息被发送到队列中。“消息队列”是在消息的传输过程中保存消息的容器。消息队列管理器在将消息从它的源中继到它的目标时充当中间人。队列的主要目的是提供路由并保证消息的传递；如果发送消息时接收者不可用，消息队列会保留消息，直到可以成功地传递它。

消息队列的主要特点是异步处理，主要目的是减少请求响应时间和解耦。所以主要的使用场景就是将比较耗时而且不需要即时（同步）返回结果的操作作为消息放入消息队列。同时由于使用了消息队列，只要保证消息格式不变，消息的发送方和接收方并不需要彼此联系，也不需要受对方的影响，即解耦和。如:

跨系统的异步通信，所有需要异步交互的地方都可以使用消息队列。就像我们除了打电话（同步）以外，还需要发短信，发电子邮件（异步）的通讯方式。

多个应用之间的耦合，由于消息是平台无关和语言无关的，而且语义上也不再是函数调用，因此更适合作为多个应用之间的松耦合的接口。基于消息队列的耦合，不需要发送方和接收方同时在线。

在企业应用集成（EAI）中，文件传输，共享数据库，消息队列，远程过程调用都可以作为集成的方法。

应用内的同步变异步，比如订单处理，就可以由前端应用将订单信息放到队列，后端应用从队列里依次获得消息处理，高峰时的大量订单可以积压在队列里慢慢处理掉。由于同步通常意味着阻塞，而大量线程的阻塞会降低计算机的性能。

消息驱动的架构（EDA），系统分解为消息队列，和消息制造者和消息消费者，一个处理流程可以根据需要拆成多个阶段（Stage），阶段之间用队列连接起来，前一个阶段处理的结果放入队列，后一个阶段从队列中获取消息继续处理。

应用需要更灵活的耦合方式，如发布订阅，比如可以指定路由规则。

跨局域网，甚至跨城市的通讯，比如北京机房与广州机房的应用程序的通信。

## 什么是ActiveMQ
ActiveMQ 是Apache出品，最流行的，能力强劲的开源消息总线。ActiveMQ 是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现,尽管JMS规范出台已经是很久的事情了,但是JMS在当今的J2EE应用中间仍然扮演着特殊的地位。

主要特点：

* 多种语言和协议编写客户端。语言: Java, C, C++, C#, Ruby, Perl, Python, PHP。应用协议: OpenWire,Stomp REST,WS Notification,XMPP,AMQP    
* 完全支持JMS1.1和J2EE 1.4规范 (持久化,XA消息,事务)    
* 对Spring的支持,ActiveMQ可以很容易内嵌到使用Spring的系统里面去,而且也支持Spring2.0的特性    
* 通过了常见J2EE服务器(如 Geronimo,JBoss 4, GlassFish,WebLogic)的测试,其中通过JCA 1.5 resource adaptors的配置,可以让ActiveMQ可以自动的部署到任何兼容J2EE 1.4 商业服务器上    
* 支持多种传送协议:in-VM,TCP,SSL,NIO,UDP,JGroups,JXTA    
* 支持通过JDBC和journal提供高速的消息持久化    
* 从设计上保证了高性能的集群,客户端-服务器,点对点    
* 支持Ajax    
* 支持与Axis的整合    
* 可以很容易得调用内嵌JMS provider,进行测试  

## ActiveMQ主要概念

### 1 Destination
目的地，JMS Provider（消息中间件）负责维护，用于对Message进行管理的对象。MessageProducer需要指定Destination才能发送消息，MessageConsumer需要指定Destination才能接收消息。

### 2 Producer
消息生成者(客户端、生成消息)，负责发送Message到目的地。应用接口为MessageProducer。在JMS规范中，所有的标准定义都在javax.jms包中。

### 3 Consumer【Receiver】
消息消费者（处理消息），负责从目的地中消费【处理|监听|订阅】Message。应用接口为MessageConsumer

### 4 Message
消息（Message），消息封装一次通信的内容。常见类型有：StreamMessage、BytesMessage、TextMessage、ObjectMessage、MapMessage。

### 5 ConnectionFactory
链接工厂, 用于创建链接的工厂类型。 注意，不能和JDBC中的ConnectionFactory混淆。

### 6 Connection
链接. 用于建立访问ActiveMQ连接的类型, 由链接工厂创建. 注意，不能和JDBC中的Connection混淆。

### 7 Session
会话, 一次持久有效有状态的访问. 由链接创建. 是具体操作消息的基础支撑。

### 8 Queue & Topic
Queue是队列目的地，Topic是主题目的地。都是Destination的子接口。

Queue特点： 队列中的消息，默认只能由唯一的一个消费者处理。一旦处理消息删除。

Topic特点：主题中的消息，会发送给所有的消费者同时处理。只有在消息可以重复处理的业务场景中可使用。

### 9 PTP
Point to Point。点对点消息模型。就是基于Queue实现的消息处理方式。

### 10 PUB & SUB
Publish & Subscribe 。消息的发布/订阅模型。是基于Topic实现的消息处理方式。

## ActiveMQ的消息形式
对于消息的传递有两种类型：

**一种是点对点**的，即一个生产者和一个消费者一一对应；
**另一种是发布/订阅模式**，即一个生产者产生消息并进行发送后，可以由多个消费者进行接收。

JMS定义了五种不同的消息正文格式，以及调用的消息类型，允许你发送并接收以一些不同形式的数据，提供现有消息格式的一些级别的兼容性。
```
• StreamMessage -- Java原始值的数据流
• MapMessage--一套名称-值对
• TextMessage--一个字符串对象
• ObjectMessage--一个序列化的 Java对象
• BytesMessage--一个字节的数据流
```

## ActiveMQ安装
### 1 下载资源
ActiveMQ官网： http://activemq.apache.org

版本说明:

ActiveMQ5.10.x以上版本必须使用JDK1.8才能正常使用。

ActiveMQ5.9.x及以下版本使用JDK1.7即可正常使用。

### 2 上传至Linux服务器

### 3 解压安装文件
tar -zxf apache-activemq-5.9.0-bin.tar.gz

### 4 检查权限
ls -al apache-activemq-5.9.0/bin

如果权限不足,则无法执行,需要修改文件权限:

chmod 755 activemq

### 5 复制应用至本地目录
cp -r apache-activemq-5.9.0 /usr/local/activemq

### 6 配置文件简介
/usr/local/activemq/conf/* - 配置文件.

需要关注的配置文件有: activemq.xml, jetty.xml, users.properties

任何配置文件修改后,必须重启ActiveMQ,才能生效.

#### 6.1 activemq.xml
就是spring配置文件. 其中配置的是ActiveMQ应用使用的默认对象组件.

transportConnectors标签 - 配置链接端口信息的. 其中的端口号61616是ActiveMQ对外发布的tcp协议访问端口. 就是java代码访问ActiveMQ时使用的端口.

#### 6.2 jetty.xml
spring配置文件, 用于配置jetty服务器的默认对象组件.

jetty是类似tomcat的一个中间件容器.

ActiveMQ默认支持一个网页版的服务查看站点. 可以实现ActiveMQ中消息相关数据的页面查看.

8161端口, 是ActiveMQ网页版管理站点的默认端口.

在ActiveMQ网页版管理站点中,需要登录, 默认的用户名和密码都是admin.

#### 6.3 users.properties
内容信息: 用户名=密码

是用于配置客户端通过协议访问ActiveMQ时,使用的用户名和密码.

### 7 启动ActiveMQ
bin/activemq start

### 8 测试ActiveMQ
#### 8.1 检查进程
ps aux | grep activemq

#### 8.2 管理界面
使用浏览器访问ActiveMQ管理应用, 地址如下:

http://ip:8161/admin/

用户名: admin

密码: admin

ActiveMQ使用的是jetty提供HTTP服务.启动稍慢,建议短暂等待再访问测试.

#### 8.3 修改访问端口
修改ActiveMQ配置文件: /usr/local/activemq/conf/jetty.xml
![](assets/markdown-img-paste-20200219005112984.png)
配置文件修改完毕，保存并重新启动ActiveMQ服务。

### 9 重启ActiveMQ
bin/activemq restart

### 10 关闭ActiveMQ
bin/activemq stop










