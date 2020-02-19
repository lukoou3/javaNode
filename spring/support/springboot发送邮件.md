# springboot发送邮件
`https://www.jianshu.com/p/5eb000544dd7`

## 原理

### 什么是JavaMailSender和JavaMailSenderImpl？
JavaMailSender和JavaMailSenderImpl 是Spring官方提供的集成邮件服务的接口和实现类，以简单高效的设计著称，目前是Java后端发送邮件和集成邮件服务的主流工具。

### 如何通过JavaMailSenderImpl发送邮件？
非常简单，直接在业务类注入JavaMailSenderImpl并调用send方法发送邮件。其中简单邮件可以通过SimpleMailMessage来发送邮件，而复杂的邮件（例如添加附件）可以借助MimeMessageHelper来构建MimeMessage发送邮件。例如：
```java
@Autowired
private JavaMailSenderImpl mailSender;

public void sendMail() throws MessagingException {
    //简单邮件
    SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
    simpleMailMessage.setFrom("admin@163.com");
    simpleMailMessage.setTo("socks@qq.com");
    simpleMailMessage.setSubject("Happy New Year");
    simpleMailMessage.setText("新年快乐！");
    mailSender.send(simpleMailMessage);

    //复杂邮件
    MimeMessage mimeMessage = mailSender.createMimeMessage();
    MimeMessageHelper messageHelper = new MimeMessageHelper(mimeMessage);
    messageHelper.setFrom("admin@163.com");
    messageHelper.setTo("socks@qq.com");
    messageHelper.setSubject("Happy New Year");
    messageHelper.setText("新年快乐！");
    messageHelper.addInline("doge.gif", new File("xx/xx/doge.gif"));
    messageHelper.addAttachment("work.docx", new File("xx/xx/work.docx"));
    mailSender.send(mimeMessage);
}
```

### 为什么JavaMailSenderImpl 能够开箱即用 ？
所谓开箱即用其实就是基于官方内置的自动配置，翻看源码可知晓邮件自动配置类(MailSenderPropertiesConfiguration) 为上下文提供了邮件服务实例(JavaMailSenderImpl)。具体源码如下：
```java
@Configuration
@ConditionalOnProperty(prefix = "spring.mail", name = "host")
class MailSenderPropertiesConfiguration {

	private final MailProperties properties;

	MailSenderPropertiesConfiguration(MailProperties properties) {
		this.properties = properties;
	}

	@Bean
	@ConditionalOnMissingBean(JavaMailSender.class)
	public JavaMailSenderImpl mailSender() {
		JavaMailSenderImpl sender = new JavaMailSenderImpl();
		applyProperties(sender);
		return sender;
	}

	private void applyProperties(JavaMailSenderImpl sender) {
		sender.setHost(this.properties.getHost());
		if (this.properties.getPort() != null) {
			sender.setPort(this.properties.getPort());
		}
		sender.setUsername(this.properties.getUsername());
		sender.setPassword(this.properties.getPassword());
		sender.setProtocol(this.properties.getProtocol());
		if (this.properties.getDefaultEncoding() != null) {
			sender.setDefaultEncoding(this.properties.getDefaultEncoding().name());
		}
		if (!this.properties.getProperties().isEmpty()) {
			sender.setJavaMailProperties(asProperties(this.properties.getProperties()));
		}
	}

	private Properties asProperties(Map<String, String> source) {
		Properties properties = new Properties();
		properties.putAll(source);
		return properties;
	}

}
```

其中MailProperties是关于邮件服务器的配置信息，具体源码如下：
```java
@ConfigurationProperties(prefix = "spring.mail")
public class MailProperties {

	private static final Charset DEFAULT_CHARSET = StandardCharsets.UTF_8;

	/**
	 * SMTP server host. For instance, `smtp.example.com`.
	 */
	private String host;

	/**
	 * SMTP server port.
	 */
	private Integer port;

	/**
	 * Login user of the SMTP server.
	 */
	private String username;

	/**
	 * Login password of the SMTP server.
	 */
	private String password;

	/**
	 * Protocol used by the SMTP server.
	 */
	private String protocol = "smtp";

	/**
	 * Default MimeMessage encoding.
	 */
	private Charset defaultEncoding = DEFAULT_CHARSET;

	/**
	 * Additional JavaMail Session properties.
	 */
	private Map<String, String> properties = new HashMap<>();

	/**
	 * Session JNDI name. When set, takes precedence over other Session settings.
	 */
	private String jndiName;
    
    ...
}
```

## springboot测试mail
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
