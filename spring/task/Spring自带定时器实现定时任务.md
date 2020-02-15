## Spring自带定时器实现定时任务
在Spring框架中实现定时任务的办法至少有2种（不包括Java原生的Timer及Executor实现方式），一种是集成第三方定时任务框架，如无处不在的Quartz；另一种便是Spring自带的定时器（仅针对3.0之后的版本）。本文说的是Spring自带定时器，看看使用起来到底有多简单。

下面介绍两种方式实现Spring定时器功能，一种是基于xml配置方式，一种是基于注解的方式。

### 基于xml配置的spring定时器
```java
public class MyTask {
  public void execute(){
    System.out.println("基于注解配置的spring定时任务！");
  }
}
```

编写个spring-task.xml专门用来配置加载定时任务信息
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
		http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-3.2.xsd"
       default-lazy-init="true">
    <task:scheduled-tasks>
        <!--直充重试机制-->
        <task:scheduled ref="myTask" method="execute" cron="0 1 * * * ?"/>  ##myTask必须是定时任务类名小写
    </task:scheduled-tasks>
</beans>
```

并把此加入spring配置文件中
```xml
<!--  task  -->
<import resource="spring-task.xml"/>
```

### 基于注解配置的spring定时器
使用两个注解：@EnableScheduling、@Scheduled来简单实现定时任务。

定时任务需要在配置类上添加@EnableScheduling，表示对定时任务的支持。

在对应执行任务的方法上添加@Scheduled，声明需要执行定时任务的方法。

Scheduled中包含以下几个参数：

* 1）cron是设置定时执行的cron表达式    
* 2）zone表示执行时间时区    
* 3）fixedDelay 和fixedDelayString 表示固定延迟时间，上个任务完成后，延迟多长时间执行    
* 4）fixedRate 和fixedRateString表示固定频率，上个任务开始后，多长时间后开始执行    
* 5）initialDelay 和initialDelayString表示初始延迟时间，第一次被调用前延迟的时间  

注解支持：
```xml
<context:component-scan base-package="com.simonsfan.study.Task">
<task:annotation-driven/>
```

代码如下：
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;

@Configuration
@EnableScheduling
public class Scheduling {
 
    @Scheduled(cron = "0 0 0/1 * * ?")
    public void minute() {
    	System.out.println("==>每天一个小时执行一次");
    }
    
    @Scheduled(cron = "* */5 * * * SUN-MON")
    public void count() {
    	System.out.println("==>周一至周五每隔5分钟执行一次");
    }
    
    // 初始时延迟3秒，每隔10秒
    @Scheduled(fixedRateString = "10000",initialDelay = 3000)
    public void fixedRate(){
    	System.out.println("==>初始延迟3秒，每隔10秒");
    }
 
    // 每次执行完，延迟10秒
    @Scheduled(fixedDelayString= "10000")
    public void fixedDelay(){
    	System.out.println("==>每次执行完延迟10秒");
    }
    
}
```