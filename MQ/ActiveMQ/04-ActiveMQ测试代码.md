# ActiveMQ测试代码

## ActiveMQ本身测试

### 依赖
引入activemq-core和activemq-client都能测试成功
```xml
<!--<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-core</artifactId>
    <version>5.7.0</version>
</dependency>-->

<!--single 中只引入这个依赖就可以-->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-client</artifactId>
    <version>5.14.5</version>
</dependency>
```

### Queue测试
#### TextProducer
```java
package com.first;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 发送一个字符串文本消息到ActiveMq中
 * @author lenovo
 *
 */
public class TextProducer {

    /**
     * 发送消息到ActiveMq中，具体消息内容为参数信息
     * 开发JMS相关代码过程中，使用的接口类型都是javax.jms包下的类型
     * @param datas - 消息内容
     */
    public void sendTextMessage(String datas) {
        //连接工厂
        ConnectionFactory factory = null;
        //连接
        Connection connection = null;
        //目的地
        Destination destination = null;
        //会话
        Session session = null;
        //消息发送者
        MessageProducer producer = null;
        //消息对象
        Message message = null;

        try {
            //创建连接工厂，连接ActiveMQ服务的连接工厂
            //创建工厂，构造方法有三个参数，分别是用户名，密码，连接地址
            //无参构造，由默认连接地址。本地连接localhost
            //单参构造，无验证模式的。没有用户的认证
            //三参构造器，有认证+指定地址。默认端口是61616。从ActiveMQ的conf/activemq.xml配置文件中查看
            factory = new ActiveMQConnectionFactory("admin", "admin",
                    "tcp://192.168.216.111:61616");
            //通过工厂，创建连接对象
            //创建连接的方法有重载，其中createConnection(String userName,String password);
            //可以在创建连接工厂时，只传递连接地址，不传递用户信息
            connection = factory.createConnection();
            //建议启动连接，消息的发送者不是必须启动连接。消息的消费者必须启动连接
            //producer在发送消息的时候，会检查是否启动了链接。如果未启动，自动启动。
            //如果有特殊的配置，建议配置完毕后再启动连接
            connection.start();

            //通过连接对象，创建会话对象。必须绑定目的地
            /**
             * 创建会话的时候，必须传递两个参数，分别代表是否支持事务 和如何确认消息处理。
             * transacted - 是否支持事务，数据类型是boolean. true-支持，false-不支持
             *  true - 支持事务，第二个参数默认无效。建议传递的数据是Session.SESSION_TRANSACTED
             *  false - 不知处事务，常用参数。第二个参数必须传递，且必须有效。
             *
             * acknowledgeMode - 如何确认消息的处理。使用确认机制实现的。
             * 	AUTO_ACKNOWLEDGE - 自动确认消息。消息的消费者处理消息后，自动确认。常用（商业开发不推荐）
             * 	CLIENT_ACKNOWLEDGE - 客户端手动确认。消息的消费者处理后，必须手工确认。
             * 	DUPS_OK_ACKNOWLEDGE - 有副本的客户端手动确认。
             * 		一个消息可以多次处理
             * 		可以降低Session的消耗，在可以容忍重复消息时使用。（不推荐使用）
             */
            session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
            //创建目的地。参数是目的地名称。是目的地的唯一标记。
            destination = session.createQueue("first-mq");

            //通过会话对象，创建消息的发送者producer
            //创建的消息发送者，发送的消息一定到指定的目的地中。
            //创建producer的时候，可以不提供目的地。在发送消息的时候制定目的地。
            producer = session.createProducer(destination);

            //创建文本消息对象，作为具体数据内容的载体。
            message = session.createTextMessage(datas);

            //使用producer,发送消息到ActiveMQ中的目的地。如果消息发送失败，抛出异常
            producer.send(message);

            System.out.println("消息已发送");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //回收资源
            if(producer != null) {//回收消息发送者
                try {
                    producer.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(session != null) {//回收会话对象
                try {
                    session.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(connection != null) {//回收连接对象
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }

    }

    public static void main(String[] args) {
        TextProducer producer = new TextProducer();
        producer.sendTextMessage("测试ActiveMQ");

    }

}

```

#### TextConsumer
```java
package com.first;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class TextConsumer {

    public String receiveTextMessage() {

        String resultCode = "";
        ConnectionFactory factory = null;
        Connection connection = null;
        Session session = null;
        Destination destination = null;
        //消息的消费者，用于接收消息的接收
        MessageConsumer consumer = null;
        Message message = null;

        try {
            factory = new ActiveMQConnectionFactory("admin", "admin",
                    "tcp://192.168.216.111:61616");
            connection = factory.createConnection();
            //消息的消费者必须启动连接，否则无法处理消息
            connection.start();
            session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
            destination = session.createQueue("first-mq");
            //创建消息消费者对象。在指定的目的地中获取消息
            consumer = session.createConsumer(destination);
            //获取消息队列中的消息。receive方法是一个主动获取消息的方法。执行一次，拉取一个消息，开发少用
            message = consumer.receive();

            //确认消息，发送处理消息确认信息。通知MQ删除对应的消息。
            message.acknowledge();

            //处理文本消息
            resultCode = ((TextMessage)message).getText();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //回收资源
            if(consumer != null) {//回收消息消费者
                try {
                    consumer.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(session != null) {//回收会话对象
                try {
                    session.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(connection != null) {//回收连接对象
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }

        return resultCode;
    }

    public static void main(String[] args) {
        TextConsumer consumer = new TextConsumer();
        String message = consumer.receiveTextMessage();
        System.out.println("message:"+message);

    }

}
```

#### Queue Listener测试
ConsumerListener：
```java
package com.listener;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 使用监听器的方式，实现消息的处理【消费】
 * @author lenovo
 *
 */
public class ConsumerListener {

    /**
     * 处理消息。
     */
    public void consumMessage() {

        ConnectionFactory factory = null;
        Connection connection = null;
        Session session = null;
        Destination destination = null;
        MessageConsumer consumer = null;
        try {
            factory = new ActiveMQConnectionFactory("admin", "admin",
                    "tcp://192.168.216.111:61616");

            connection = factory.createConnection();

            connection.start();

            session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

            destination = session.createQueue("test-listener");

            consumer = session.createConsumer(destination);

            //超时，连接超时。不是确认超时，是等待多久后，没有消息可处理，超时
            //consumer.receive(timeout);

            //注册监听器。注册成功后，队列中的消息变化会自动触发监听器代码。接收消息并处理
            consumer.setMessageListener(new MessageListener() {

                /**
                 * 监听器一旦注册，永久有效。
                 * 永久 - consumer线程不关闭。
                 * 处理消息的方式：只要有消息未处理，自动调用onMessage方法，处理消息。
                 * 监听器可以注册若干。注册多个监听器，相当于集群。
                 * ActiveMQ自动的循环调用多个监听器，处理队列中的消息，实现并行处理。
                 *
                 * 处理消息的方法，就是监听方法。
                 * 监听事件是：消息，消息未处理。
                 * 要处理的具体内容：消息处理。
                 * @Param message - 未处理的消息。
                 * (non-Javadoc)
                 * @see javax.jms.MessageListener#onMessage(javax.jms.Message)
                 */
                //@Override
                public void onMessage(Message message) {
                    try {
                        //acknowledge方法，就是确认的方法。代表consumer已经收到消息。确定后，MQ删除对应的消息
                        message.acknowledge();
                        ObjectMessage om = (ObjectMessage)message;
                        Object data = om.getObject();
                        System.out.println(data);
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }

                }
            });

            //阻塞当前代码。保证listener代码未结束。如果代码结束了，监听器自动关闭
            System.in.read();

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //回收资源
            if(consumer != null) {//回收消息消费者
                try {
                    consumer.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(session != null) {//回收会话对象
                try {
                    session.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(connection != null) {//回收连接对象
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {

        ConsumerListener listener = new ConsumerListener();
        listener.consumMessage();
        System.out.println("aaaaaaa");
    }

}

```

ObjectProduce：
```java
package com.listener;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.util.Random;

public class ObjectProduce {

    public void sendMessage(Object obj) {
        ConnectionFactory factory = null;
        Connection connection = null;
        Session session = null;
        Destination destination = null;
        MessageProducer producer = null;
        Message message = null;

        try {
            factory = new ActiveMQConnectionFactory("admin", "admin",
                    "tcp://192.168.216.111:61616");

            connection = factory.createConnection();

            session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

            destination = session.createQueue("test-listener");

            producer = session.createProducer(destination);

            connection.start();

            Random random = new Random();
            for(int i = 0;i < 100;i++) {
                //Integer data = random.nextInt(100);
                Integer data = i;
                //创建对象消息，消息中的数据载体是一个可序列化的对象
                message = session.createObjectMessage(data);
                producer.send(message);
            }
        } catch (Exception e) {

            e.printStackTrace();
        }finally {
            //回收资源
            if(producer != null) {//回收消息发送者
                try {
                    producer.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(session != null) {//回收会话对象
                try {
                    session.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(connection != null) {//回收连接对象
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        ObjectProduce produce = new ObjectProduce();
        produce.sendMessage(null);

    }

}

```

### Topic测试

#### TopicProducer
```java
package com.topic;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 发送一个字符串文本消息到ActiveMQ中
 * @author lenovo
 *
 */
public class TopicProducer {

    /**
     * 发送消息到ActiveMq中，具体消息内容为参数信息
     * 开发JMS相关代码过程中，使用的接口类型都是javax.jms包下的类型
     * @param datas - 消息内容
     */
    public void sendTextMessage(String datas) {
        //连接工厂
        ConnectionFactory factory = null;
        //连接
        Connection connection = null;
        //目的地
        Destination destination = null;
        //会话
        Session session = null;
        //消息发送者
        MessageProducer producer = null;
        //消息对象
        Message message = null;

        try {
            //创建连接工厂，连接ActiceMQ服务的连接工厂。
            //创建工厂，构造方法有三个参数，分别是用户名、密码、连接地址
            factory = new ActiveMQConnectionFactory("admin", "admin",
                    "tcp://192.168.216.111:61616");

            //通过工厂，创建连接对象。
            //创建连接的方法有重载，其中createConnection(String username,String);
            //可以在创建连接工厂时，只传递连接地址，不传递用户信息。
            connection = factory.createConnection();
            //建议启动连接，消息的发送者不是必须启动；连接。消息的消费者必须启动连接。
            connection.start();

            //通过连接对象，创建会话对象。必须绑定目的地
            /**
             * 创建会话的时候，必须传递两个参数，分别代表是否支持事务 和如何确认消息处理。
             * transacted - 是否支持事务，数据类型是boolean. true-支持，false-不支持
             *  true - 支持事务，第二个参数默认无效。建议传递的数据是Session.SESSION_TRANSACTED
             *  false - 不知处事务，常用参数。第二个参数必须传递，且必须有效。
             *
             * acknowledgeMode - 如何确认消息的处理。使用确认机制实现的。
             * 	AUTO_ACKNOWLEDGE - 自动确认消息。消息的消费者处理消息后，自动确认。常用（商业开发不推荐）
             * 	CLIENT_ACKNOWLEDGE - 客户端手动确认。消息的消费者处理后，必须手工确认。
             * 	DUPS_OK_ACKNOWLEDGE - 有副本的客户端手动确认。
             * 		一个消息可以多次处理
             * 		可以降低Session的消耗，在可以容忍重复消息时使用。（不推荐使用）
             */
            session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
            //创建主题目的地。topic
            destination = session.createTopic("test-topic");

            //通过会话对象，创建消息的发送者producer
            //创建的消息发送者，发送的消息一定到指定的目的地中。
            producer = session.createProducer(destination);

            //创建文本消息对象，作为具体数据内容的载体。
            message = session.createTextMessage(datas);

            //使用producer,发送消息到ActiveMQ中的目的地。
            producer.send(message);

            System.out.println("消息已发送");

        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            //回收资源
            if(producer != null) {//回收消息发送者
                try {
                    producer.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(session != null) {//回收会话对象
                try {
                    session.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(connection != null) {//回收连接对象
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        TopicProducer topicProducer = new TopicProducer();
        topicProducer.sendTextMessage("测试ActiveMQ-4");

    }

}

```

#### TopicConsumer
```java
package com.topic;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 消息消费者
 * @author lenovo
 *
 */
public class TopicConsumer {

    public String receiveTextMessage() {
        String resultCode = "";
        ConnectionFactory factory = null;
        Connection connection = null;
        Session session = null;
        Destination destination = null;
        //消息的消费者，用于接收消息的接收
        MessageConsumer consumer = null;
        Message message = null;

        try {
            factory = new ActiveMQConnectionFactory("admin", "admin",
                    "tcp://192.168.216.111:61616");
            connection = factory.createConnection();
            //消息的消费者必须启动连接，否则无法处理消息
            connection.start();
            session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

            destination = session.createTopic("test-topic");
            //创建消息消费者对象。在指定的目的地中获取消息
            consumer = session.createConsumer(destination);
            //获取消息队列中的消息。receive方法是一个主动获取消息的方法。执行一次，拉取一个消息，开发少用
            message = consumer.receive();

            //处理文本消息
            resultCode = ((TextMessage)message).getText();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //回收资源
            if(consumer != null) {//回收消息消费者
                try {
                    consumer.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(session != null) {//回收会话对象
                try {
                    session.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(connection != null) {//回收连接对象
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }

        return resultCode;
    }

    public static void main(String[] args) {
        TopicConsumer consumer = new TopicConsumer();
        String messageString = consumer.receiveTextMessage();

        System.out.println("消息的内容是："+messageString);

    }

}

```

#### Topic Listener测试
```java
package com.topic;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * 使用监听器的方式，实现消息的处理【消费】
 * @author lenovo
 *
 */
public class TopicConsumerListener {

    /**
     * 处理消息。
     */
    public void consumMessage() {

        ConnectionFactory factory = null;
        Connection connection = null;
        Session session = null;
        Destination destination = null;
        MessageConsumer consumer = null;
        try {
            factory = new ActiveMQConnectionFactory("admin", "admin",
                    "tcp://192.168.216.111:61616");

            connection = factory.createConnection();
            connection.start();
            session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

            destination = session.createTopic("test-topic");
            //创建消息消费者对象。在指定的目的地中获取消息
            consumer = session.createConsumer(destination);


            //注册监听器。注册成功后，队列中的消息变化会自动触发监听器代码。接收消息并处理
            consumer.setMessageListener(new MessageListener() {

                /**
                 * 监听器一旦注册，永久有效。
                 * 永久 - consumer线程不关闭。
                 * 处理消息的方式：只要有消息未处理，自动调用onMessage方法，处理消息。
                 * 监听器可以注册若干。注册多个监听器，相当于集群。
                 * ActiveMQ自动的循环调用多个监听器，处理队列中的消息，实现并行处理。
                 *
                 * 处理消息的方法，就是监听方法。
                 * 监听事件是：消息，消息未处理。
                 * 要处理的具体内容：消息处理。
                 * @Param message - 未处理的消息。
                 * (non-Javadoc)
                 * @see javax.jms.MessageListener#onMessage(javax.jms.Message)
                 */
                //@Override
                public void onMessage(Message message) {
                    try {
                        //acknowledge方法，就是确认的方法。代表consumer已经收到消息。确定后，MQ删除对应的消息
                        message.acknowledge();
                        String text = ((TextMessage) message).getText();
                        System.out.println(text);
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }

                }
            });

            //阻塞当前代码。保证listener代码未结束。如果代码结束了，监听器自动关闭
            System.in.read();

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //回收资源
            if(consumer != null) {//回收消息消费者
                try {
                    consumer.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(session != null) {//回收会话对象
                try {
                    session.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if(connection != null) {//回收连接对象
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {

        TopicConsumerListener listener = new TopicConsumerListener();
        listener.consumMessage();
    }

}

```


## ActiveMQ整合spring测试


### 依赖
**不用pool时activemq只需要引入activemq-client就行了；用pool时activemq需要引入activemq-all、activemq-pool并且spring的版本从5.x改成4.x（5.x和activemq-all有冲突）。**

只有在producer使用pool，consumer使用无意义。
```xml
<!--<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-core</artifactId>
    <version>5.7.0</version>
</dependency>-->

<!--single 中只引入这个依赖就可以-->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-client</artifactId>
    <version>5.14.5</version>
</dependency>

<!--pool 中引入activemq-all、activemq-pool spring改成4.3.10.RELEASE-->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.4</version>
</dependency>


<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-pool</artifactId>
    <version>5.14.5</version>
</dependency>


<dependency>
    <groupId>javax.jms</groupId>
    <artifactId>javax.jms-api</artifactId>
    <version>2.0.1</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${spring.version}</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jms</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>${spring.version}</version>
</dependency>
```

### Queue测试
#### producer
applicationContext-jms-producer_queue.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.spring"></context:component-scan>

    <!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://192.168.216.111:61616" />
        <property name="userName" value="admin"/>
        <property name="password" value="admin"/>
    </bean>

    <!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
    <bean id="connectionFactory"
          class="org.springframework.jms.connection.SingleConnectionFactory">
        <!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
        <property name="targetConnectionFactory" ref="targetConnectionFactory" />
    </bean>

    <!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->
        <property name="connectionFactory" ref="connectionFactory" />
    </bean>

    <!--这个是队列目的地，点对点的 文本信息 -->
    <bean id="queueTextDestination" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg value="queue_text" />
    </bean>

</beans>
```

QueueProducer
```java
package com.spring.single;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;
import org.springframework.stereotype.Component;

import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Session;

@Component
public class QueueProducer {

    @Autowired
    private JmsTemplate jmsTemplate;

    @Autowired
    private Destination queueTextDestination;

    /**
     * 发送文本消息
     *
     * @param text
     */
    public void sendTextMessage(final String text) {

        jmsTemplate.send(queueTextDestination, new MessageCreator() {

            @Override
            public Message createMessage(Session session) throws JMSException {

                return session.createTextMessage(text);
            }
        });
    }
}
```

QueueProducerTest
```java
package com.spring.single;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext-jms-producer_queue.xml")
public class QueueProducerTest {
    @Autowired
    private QueueProducer queueProducer;

    @Test
    public void sendTextMessage() {
        queueProducer.sendTextMessage("SpringJms-queue");
    }
}
```


#### consumer
applicationContext-jms-consumer-queue.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://192.168.216.111:61616" />
        <property name="userName" value="admin"/>
        <property name="password" value="admin"/>
    </bean>

    <!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
    <bean id="connectionFactory"
          class="org.springframework.jms.connection.SingleConnectionFactory">
        <!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
        <property name="targetConnectionFactory" ref="targetConnectionFactory" />
    </bean>

    <!--这个是队列目的地，点对点的 文本信息 -->
    <bean id="queueTextDestination" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg value="queue_text" />
    </bean>

    <!-- 我的监听类 -->
    <bean id="myQueueMessageListener" class="com.spring.single.MyQueueMessageListener"></bean>
    <!-- 消息监听容器 -->
    <bean class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="destination" ref="queueTextDestination" />
        <property name="messageListener" ref="myQueueMessageListener" />
    </bean>

</beans>
```

MyQueueMessageListener
```java
package com.spring.single;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class MyQueueMessageListener implements MessageListener {

    public void onMessage(Message message) {
        TextMessage textMessage = (TextMessage) message;
        try {
            System.out.println("接收到消息：" + textMessage.getText());
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}

```

MyQueueMessageListenerTest
```java
package com.spring.single;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.io.IOException;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext-jms-consumer-queue.xml")
public class MyQueueMessageListenerTest {

    @Test
    public void test() {
        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### Topic测试

#### producer
applicationContext-jms-producer_topic.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">


    <context:component-scan base-package="com.spring"></context:component-scan>


    <!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://192.168.216.111:61616" />
        <property name="userName" value="admin"/>
        <property name="password" value="admin"/>
    </bean>

    <!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
    <bean id="connectionFactory"
          class="org.springframework.jms.connection.SingleConnectionFactory">
        <!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
        <property name="targetConnectionFactory" ref="targetConnectionFactory" />
    </bean>

    <!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->
        <property name="connectionFactory" ref="connectionFactory" />
    </bean>


    <!--这个是订阅模式 文本信息 -->
    <bean id="topicTextDestination" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg value="topic_text" />
    </bean>

</beans>
```

TopicProducer
```java
package com.spring.single;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;
import org.springframework.stereotype.Component;

import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Session;

@Component
public class TopicProducer {
    @Autowired
    private JmsTemplate jmsTemplate;

    @Autowired
    private Destination topicTextDestination;

    /**
     * 发送文本消息
     *
     * @param text
     */
    public void sendTextMessage(final String text) {
        jmsTemplate.send(topicTextDestination, new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage(text);
            }
        });
    }
}

```

TopicProducerTest
```java
package com.spring.single;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext-jms-producer_topic.xml")
public class TopicProducerTest {
    @Autowired
    private TopicProducer topicProducer;

    @Test
    public void sendTextMessage() {
        for (int i = 0; i < 10; i++) {
            topicProducer.sendTextMessage("SpringJms-topic" + i);
        }
    }
}
```

#### consumer
applicationContext-jms-consumer-topic.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://192.168.216.111:61616" />
        <property name="userName" value="admin"/>
        <property name="password" value="admin"/>
    </bean>

    <!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
    <bean id="connectionFactory"
          class="org.springframework.jms.connection.SingleConnectionFactory">
        <!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
        <property name="targetConnectionFactory" ref="targetConnectionFactory" />
    </bean>

    <!--这个是队列目的地，点对点的 文本信息 -->
    <bean id="topicTextDestination" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg value="topic_text" />
    </bean>

    <!-- 我的监听类 -->
    <bean id="myTopicMessageListener" class="com.spring.single.MyTopicMessageListener"></bean>
    <!-- 消息监听容器 -->

    <bean class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="destination" ref="topicTextDestination" />
        <property name="messageListener" ref="myTopicMessageListener" />
    </bean>

</beans>
```

MyTopicMessageListener
```java
package com.spring.single;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class MyTopicMessageListener implements MessageListener {
    public void onMessage(Message message) {
        TextMessage textMessage = (TextMessage) message;
        try {
            System.out.println("接收到消息：" + textMessage.getText());
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}
```

MyTopicMessageListenerTest
```java
package com.spring.single;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.io.IOException;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext-jms-consumer-topic.xml")
public class MyTopicMessageListenerTest {

    @Test
    public void test() {
        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### pool测试
只有在producer使用pool，consumer使用无意义。

#### producer
applicationContext-jms-pool-producer.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jms="http://www.springframework.org/schema/jms"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd
    http://www.springframework.org/schema/jms
	http://www.springframework.org/schema/jms/spring-jms.xsd
	http://activemq.apache.org/schema/core
	http://activemq.apache.org/schema/core/activemq-core.xsd
	">

    <context:component-scan base-package="com.spring.pool"></context:component-scan>

    <!-- 配置ActiveMQConnectionFactory对象。 -->
    <!--<amq:connectionFactory brokerURL="tcp://192.168.216.111:61616"
                           userName="admin" password="admin" id="amqConnectionFactory">
    </amq:connectionFactory>-->

    <!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://192.168.216.111:61616" />
        <property name="userName" value="admin"/>
        <property name="password" value="admin"/>
    </bean>

    <!-- 配置池化的ConnectionFactory。为连接ActiveMQ的ConnectionFactory提供连接池 -->
    <!--PooledConnectionFactory会缓存connection，session，和producer，不会缓存consumer，更适合于发送者。-->
    <bean id="pooledConnectionFactory"
          class="org.apache.activemq.pool.PooledConnectionFactoryBean">
        <property name="connectionFactory" ref="targetConnectionFactory"></property>
        <property name="maxConnections" value="10"></property>
    </bean>

    <!-- 配置有缓存的connectionFactory，session的缓存大小可以定制。 -->
    <bean id="connectionFactory"
          class="org.springframework.jms.connection.CachingConnectionFactory">
        <!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
        <property name="targetConnectionFactory" ref="pooledConnectionFactory"></property>
        <!-- Session缓存数量 ，设置一次性可以发或者接收多少个消息-->
        <property name="sessionCacheSize" value="3"></property>
    </bean>

    <!-- JmsTemplate配置 -->
    <bean id="template" class="org.springframework.jms.core.JmsTemplate">
        <!-- 给定连接工厂，必须是spring创建的连接工厂 -->
        <property name="connectionFactory" ref="connectionFactory"></property>
        <!-- 可选 - 默认目的地命名 -->
        <property name="defaultDestinationName" value="test-spring-pool"></property>
    </bean>

</beans>
```

PoolQueueProducer
```java
package com.spring.pool;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;
import org.springframework.stereotype.Component;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Session;

@Component
public class PoolQueueProducer {

    @Autowired
    private JmsTemplate jmsTemplate;


    /**
     * 发送文本消息
     *
     * @param text
     */
    public void sendTextMessage(final String text) {

        jmsTemplate.send(new MessageCreator() {

            @Override
            public Message createMessage(Session session) throws JMSException {

                return session.createTextMessage(text);
            }
        });
    }
}
```

PoolQueueProducerTest
```java
package com.spring.pool;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext-jms-pool-producer.xml")
public class PoolQueueProducerTest {
    @Autowired
    private PoolQueueProducer queueProducer;

    @Test
    public void sendTextMessage() {
        for (int i = 0; i < 5; i++) {
            queueProducer.sendTextMessage("SpringJms-pool-queue" + i);
        }

    }
}
```


#### consumer
applicationContext-jms-pool-consumer.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jms="http://www.springframework.org/schema/jms"
       xmlns:amq="http://activemq.apache.org/schema/core"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd
    http://www.springframework.org/schema/jms
	http://www.springframework.org/schema/jms/spring-jms.xsd
	http://activemq.apache.org/schema/core
	http://activemq.apache.org/schema/core/activemq-core.xsd
	">

    <!-- 配置service的扫描和jms相关资源 -->

    <!-- 配置ActiveMQConnectionFactory对象。 -->
    <amq:connectionFactory brokerURL="tcp://192.168.216.111:61616"
                           userName="admin" password="admin" trustAllPackages="true" id="amqConnectionFactory"></amq:connectionFactory>

    <!-- spring管理JMS相关代码的时候，必须依赖jms标签库，spring-jms提供的标签库 -->
    <!-- 定义spring-jms中的连接工厂
        CachingConnectionFactory - spring框架提供的连接工厂对象，不能真正的访问mom容器。
        类似一个工厂的代理对象，需要提供一个真实工厂，实现mom容器的连接访问。
    -->
    <bean id="connectionFactory"
          class="org.springframework.jms.connection.CachingConnectionFactory">
        <!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
        <property name="targetConnectionFactory" ref="amqConnectionFactory"></property>
        <!-- Session缓存数量 ，设置一次性可以发或者接收多少个消息-->
        <property name="sessionCacheSize" value="3"></property>
    </bean>


    <!-- 注册监听器 -->
    <!-- 开始注册监听：
        需要的参数有：
        acknowledge - 消息确认机制
        container-type - 容器类型，默认为
        DefaultContainerType SingleContainerType
        destination-type - 目的地类型，使用队列作为目的地。
        connection-factory - 连接工厂，spring-jms使用的连接工厂，必须是spring自主
        创建的，不能使用第三方创建的工程。如：ActiveMQConnectionFactory. -->
    <jms:listener-container acknowledge="auto" container-type="default"
                            destination-type="queue" connection-factory="connectionFactory">
        <!-- 监听容器中注册的某监听对象
            destination - 设置目的地命名 -->
        <jms:listener destination="test-spring-pool" ref="myListener"/>
    </jms:listener-container>

    <bean id="myListener" class="com.spring.pool.MyPoolMessageListener"/>
</beans>
```

MyPoolMessageListener
```java
package com.spring.pool;

import org.springframework.stereotype.Component;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

@Component(value="myListener")
public class MyPoolMessageListener implements MessageListener {

    /**
     * 监听方法
     */
    public void onMessage(Message message) {
        TextMessage textMessage = (TextMessage) message;
        try {
            System.out.println("接收到消息：" + textMessage.getText());
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }

}
```

MyPoolMessageListenerTest
```java
package com.spring.pool;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.io.IOException;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext-jms-pool-consumer.xml")
public class MyPoolMessageListenerTest {

    @Test
    public void test() {
        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```










```java

```

```xml

```





