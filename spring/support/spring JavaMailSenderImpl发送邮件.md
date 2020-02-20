# spring JavaMailSenderImpl发送邮件
`https://www.iteye.com/blog/trinea-1278334`

本文主要介绍利用**JavaMailSenderImpl**发送邮件。首先介绍了发送**一般邮件**，然后介绍了发送**富文本（html）邮件**。

 
邮件发送分为为三步：**创建邮件发送器**、**编写邮件**、**发送邮件**。

Spring的JavaMailSenderImpl提供了强大的邮件发送功能，可发送普通文本邮件、带附件邮件、html格式邮件、带图片邮件、设置发送内容编码格式、设置发送人的显示名称。

 
下面就进行介绍，示例代码中很多都是字符串硬编码，实际使用时推荐使用spring的配置文件进行配置。

## 1、创建邮件发送器
首先定义JavaMailSenderImpl对象，并对其进行smtp相关信息设置，相当于我们自己的邮箱，如下：
```java
JavaMailSenderImpl mailSender = new JavaMailSenderImpl();
mailSender.setHost("smtp.qq.com");
mailSender.setUsername("mosaic@qq.com");
mailSender.setPassword("asterisks");
```
当然更好的方法是使用配置文件进行配置，这里只是进行介绍，忽略硬编码先。

以上主机为邮箱服务商的smtp地址，用户名、密码为用户自己的邮箱。除以上外还可以设置

setPort(int port) 、setProtocol(String protocol) 等，可暂时不考虑。

这样我们便类似创建好了一个邮件发送器


## 2、 开始写邮件，编写邮件内容
JavaMailSenderImpl支持MimeMessages和SimpleMailMessages。

MimeMessages为复杂邮件模板，支持文本、附件、html、图片等。

SimpleMailMessages实现了MimeMessageHelper，为普通邮件模板，支持文本。

下面先以SimpleMailMessages为例进行介绍
```java
SimpleMailMessage smm = new SimpleMailMessage();
// 设定邮件参数
smm.setFrom(mailSender.getUsername());
smm.setTo("mosaic@126.com");
smm.setSubject("Hello world");
smm.setText("Hello world via spring mail sender");
// 发送邮件
mailSender.send(smm);
```

如此，我们便完成了一个简单邮件的编写，对于**复杂邮件**，编写及发送如下
```java
//使用JavaMail的MimeMessage，支付更加复杂的邮件格式和内容
MimeMessage msg = mailSender.createMimeMessage();
//创建MimeMessageHelper对象，处理MimeMessage的辅助类
MimeMessageHelper helper = new MimeMessageHelper(msg, true);
//使用辅助类MimeMessage设定参数
helper.setFrom(mailSender.getUsername());
helper.setTo("mosaic@126.com");
helper.setSubject("Hello Attachment");
helper.setText("This is a mail with attachment");
//加载文件资源，作为附件
ClassPathResource file = new ClassPathResource("Chrysanthemum.jpg");
//加入附件
helper.addAttachment("attachment.jpg", file);
// 发送邮件
mailSender.send(smm);
```

其中MimeMessageHelper为的辅助类MimeMessages。以上包含了以资源文件为附件进行发送。对于普通文件发送方式如下：
```java
FileSystemResource file = new FileSystemResource("C:\\Users\\image1.jpg");
helper.addInline("file", file);
```

## 3、发送邮件
2中已经包含了发送的代码，只需使用JavaMailSenderImpl的send接口即可。支持类型为
```java
void	send(MimeMessage mimeMessage) 
         Send the given JavaMail MIME message.
void	send(MimeMessage[] mimeMessages) 
         Send the given array of JavaMail MIME messages in batch.
void	send(MimeMessagePreparator mimeMessagePreparator) 
         Send the JavaMail MIME message prepared by the given MimeMessagePreparator.
void	send(MimeMessagePreparator[] mimeMessagePreparators) 
         Send the JavaMail MIME messages prepared by the given MimeMessagePreparators.
void	send(SimpleMailMessage simpleMessage) 
         Send the given simple mail message.
void	send(SimpleMailMessage[] simpleMessages) 
         Send the given array of simple mail messages in batch.
```

## 4、发送html文件
只需要在MimeMessageHelper setText时将是否是html设为true即可。setText介绍如下：
```java
setText(String text, boolean html) 
          Set the given text directly as content in non-multipart mode or as default body part in multipart mode.
```

## 测试

### 依赖
```xml
        <!-- spring email 依赖这个-->
        <dependency>
            <groupId>com.sun.mail</groupId>
            <artifactId>javax.mail</artifactId>
            <version>1.6.2</version>
        </dependency>


        <!--spring email 测试-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--包含支持缓存Cache（ehcache）、JCA、JMX、 邮件服务（Java Mail、COS Mail）、任务计划Scheduling（Timer、Quartz）方面的类。-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>
        
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>compile</scope>
        </dependency>
```

### 配置文件
mail-config.properties：
```properties
# qq mail server
#mail.protocol=smtp
#mail.port=465
#mail.host=smtp.exmail.qq.com
#mail.username=xxx@qq.com
#mail.password=

# 163 mail server
mail.protocol=smtp
mail.port=465
mail.host=smtp.163.com
mail.username=18638489474@163.com
# 163输的是授权码
mail.password=l123456
mail.defaultEncoding=UTF-8
```

applicationContext-mail.xml：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 注解注入bean -->
    <context:component-scan base-package="com.springemail"></context:component-scan>


    <context:property-placeholder location="classpath:mail-config.properties"/>

    <!-- spring mail  -->
    <!-- 注意:这里的参数(如用户名、密码)都是针对邮件发送者的 -->
    <bean id="sender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
        <property name="host" value="${mail.host}"></property>
        <property name="username" value="${mail.username}"></property>
        <property name="password" value="${mail.password}"></property>
        <property name="defaultEncoding" value="${mail.defaultEncoding}"></property>
        <property name="javaMailProperties">
            <props>
                <prop key="mail.smtp.auth">true</prop>
                <prop key="mail.smtp.timeout">30000</prop>
                <!--<prop key="mail.smtp.socketFactory.class">javax.net.ssl.SSLSocketFactory</prop>-->
                <!-- 如果是网易邮箱， mail.smtp.starttls.enable 设置为 false-->
                <!--<prop key="mail.smtp.starttls.enable">true</prop>-->
            </props>
        </property>
    </bean>

    <bean id="mailMessage" class="org.springframework.mail.SimpleMailMessage" scope="prototype">
        <property name="from" value="${mail.username}"></property>
    </bean>

    <!--<bean id="pool" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
        &lt;!&ndash; 初始容量，核心容量，最小线程数 &ndash;&gt;
        <property name="corePoolSize" value="4"></property>
        &lt;!&ndash; 空闲时间，单位秒 &ndash;&gt;
        <property name="keepAliveSeconds" value="60"></property>
        &lt;!&ndash; 最大容量 &ndash;&gt;
        <property name="maxPoolSize" value="32"></property>
        &lt;!&ndash; 任务队列容量 &ndash;&gt;
        <property name="queueCapacity" value="256"></property>
    </bean>-->

</beans>
```

### 测试代码
```java
package com.springemail;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSenderImpl;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext-mail.xml")
public class SpringEmailTest {
    @Autowired
    private JavaMailSenderImpl mailSender;

    @Test
    // 发送文本
    public void testSimpleMailMessage() {
        SimpleMailMessage mailMessage = new SimpleMailMessage();

        // 设定邮件参数
        mailMessage.setFrom(mailSender.getUsername());
        mailMessage.setTo("904319017@qq.com");
        mailMessage.setSubject("Hello world");
        mailMessage.setText("Hello world via spring mail sender");

        // 发送邮件
        mailSender.send(mailMessage);
    }

    @Test
    // 带附件
    public void testMimeMessage() throws MessagingException {
        //使用JavaMail的MimeMessage，支付更加复杂的邮件格式和内容
        MimeMessage msg = mailSender.createMimeMessage();
        //创建MimeMessageHelper对象，处理MimeMessage的辅助类
        MimeMessageHelper helper = new MimeMessageHelper(msg, true);
        //使用辅助类MimeMessage设定参数
        helper.setFrom(mailSender.getUsername());
        helper.setTo("904319017@qq.com");
        helper.setSubject("Hello Attachment");
        helper.setText("This is a mail with attachment");
        //从classpath加载文件资源，作为附件
        ClassPathResource file = new ClassPathResource("background.jpg");

        // 从文件系统
        FileSystemResource file2 = new FileSystemResource("D:\\img_tmp\\background.jpg");
        helper.addAttachment("file2.jpg", file2);

        //加入附件
        helper.addAttachment("background.jpg", file);
        // 发送邮件
        mailSender.send(msg);
    }

    @Test
    // 发送html，只需要在MimeMessageHelper setText时将是否是html设为true即可。
    public void testMimeMessageHtml() throws MessagingException {
        //使用JavaMail的MimeMessage，支付更加复杂的邮件格式和内容
        MimeMessage msg = mailSender.createMimeMessage();
        //创建MimeMessageHelper对象，处理MimeMessage的辅助类
        MimeMessageHelper helper = new MimeMessageHelper(msg, true);
        //使用辅助类MimeMessage设定参数
        helper.setFrom(mailSender.getUsername());
        helper.setTo("904319017@qq.com");
        helper.setSubject("Hello Attachment");

        //第二个参数true，表示text的内容为html
        //注意<img/>标签，src='cid:file'，'cid'是contentId的缩写，'file'是一个标记，需要在后面的代码中调用MimeMessageHelper的addInline方法替代成文件
        helper.setText("<body><p>Hello Html Email</p><img src='cid:file'/></body>", true);
        FileSystemResource file = new FileSystemResource("D:\\img_tmp\\background.jpg");
        helper.addInline("file", file);

        // 发送邮件
        mailSender.send(msg);
    }

}
```

## springboot测试
代码和spring的一样，只是xml变成了yml

### 依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```


### 配置
```yml
server:
  port: 8080

#邮件配置
spring:
  mail:
    host: smtp.163.com
    username: 18638489474@163.com
    password: l123456
    default-encoding: UTF-8
    properties:
      mail.smtp.auth: true
      mail.smtp.timeout: 3000
      mail.smtp.ssl.enable: true
```



### 测试代码
```java
package com.springboot;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSenderImpl;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.test.context.junit4.SpringRunner;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

@RunWith(SpringRunner.class)
@SpringBootTest
public class MailTest {
    @Autowired
    private JavaMailSenderImpl mailSender;

    @Test
    // 发送文本
    public void testSimpleMailMessage() {
        SimpleMailMessage mailMessage = new SimpleMailMessage();

        // 设定邮件参数
        mailMessage.setFrom(mailSender.getUsername());
        mailMessage.setTo("904319017@qq.com");
        mailMessage.setSubject("Hello world");
        mailMessage.setText("Hello world via spring mail sender");

        // 发送邮件
        mailSender.send(mailMessage);
    }

    @Test
    // 带附件
    public void testMimeMessage() throws MessagingException {
        //使用JavaMail的MimeMessage，支付更加复杂的邮件格式和内容
        MimeMessage msg = mailSender.createMimeMessage();
        //创建MimeMessageHelper对象，处理MimeMessage的辅助类
        MimeMessageHelper helper = new MimeMessageHelper(msg, true);
        //使用辅助类MimeMessage设定参数
        helper.setFrom(mailSender.getUsername());
        helper.setTo("904319017@qq.com");
        helper.setSubject("Hello Attachment");
        helper.setText("This is a mail with attachment");
        //从classpath加载文件资源，作为附件
        ClassPathResource file = new ClassPathResource("background.jpg");

        // 从文件系统
        FileSystemResource file2 = new FileSystemResource("D:\\img_tmp\\background.jpg");
        helper.addAttachment("file2.jpg", file2);

        //加入附件
        helper.addAttachment("background.jpg", file);
        // 发送邮件
        mailSender.send(msg);
    }

    @Test
    // 发送html，只需要在MimeMessageHelper setText时将是否是html设为true即可。
    public void testMimeMessageHtml() throws MessagingException {
        //使用JavaMail的MimeMessage，支付更加复杂的邮件格式和内容
        MimeMessage msg = mailSender.createMimeMessage();
        //创建MimeMessageHelper对象，处理MimeMessage的辅助类
        MimeMessageHelper helper = new MimeMessageHelper(msg, true);
        //使用辅助类MimeMessage设定参数
        helper.setFrom(mailSender.getUsername());
        helper.setTo("904319017@qq.com");
        helper.setSubject("Hello Attachment");

        //第二个参数true，表示text的内容为html
        //注意<img/>标签，src='cid:file'，'cid'是contentId的缩写，'file'是一个标记，需要在后面的代码中调用MimeMessageHelper的addInline方法替代成文件
        helper.setText("<body><p>Hello Html Email</p><img src='cid:file'/></body>", true);
        FileSystemResource file = new FileSystemResource("D:\\img_tmp\\background.jpg");
        helper.addInline("file", file);

        // 发送邮件
        mailSender.send(msg);
    }
}

```





```java

```
