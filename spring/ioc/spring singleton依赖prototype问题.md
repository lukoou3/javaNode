# spring singleton依赖prototype问题
在使用Spring时，可能会遇到这种情况：一个单例的Bean依赖另一个非单例的Bean。如果简单的使用自动装配来注入依赖，多例是不生效的。如下所示：

单例的Class A
```java
@Component
public class ClassA {
 @Autowired
 private ClassB classB;
 
 public void printClass() {
  System.out.println("This is Class A: " + this);
  classB.printClass();
 }
}
```

非单例的Class B
```java
@Component
@Scope(value = SCOPE_PROTOTYPE)
public class ClassB {
  public void printClass() {
    System.out.println("This is Class B: " + this);
  }
}
```

这里Class A采用了默认的单例scope，并依赖于Class B, 而Class B的scope是prototype，因此不是单例的，这时候跑个测试就看出这样写的问题：
```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = {ClassA.class, ClassB.class})
public class MyTest {
  @Autowired
  private ClassA classA;
 
  @Test
  public void simpleTest() {
    for (int i = 0; i < 3; i++) {
      classA.printClass();
    }
  }
}
```

输出的结果是：
```java
This is Class A: ClassA@282003e1
This is Class B: ClassB@7fad8c79
This is Class A: ClassA@282003e1
This is Class B: ClassB@7fad8c79
This is Class A: ClassA@282003e1
This is Class B: ClassB@7fad8c79
```

可以看到，两个类的Hash Code在三次输出中都是一样。Class A的值不变是可以理解的，因为它是单例的，但是Class B的scope是prototype却也保持Hash Code不变，似乎也成了单例？

产生这种的情况的原因是，Class A的scope是默认的singleton，因此Context只会创建Class A的bean一次，所以也就只有一次注入依赖的机会，容器也就无法每次给Class A提供一个新的Class B。

## 实现ApplicationContextAware(不那么好的解决方案)
要解决上述问题，可以对Class A做一些修改，让它实现ApplicationContextAware。
```java
@Component
public class ClassA implements ApplicationContextAware {
  private ApplicationContext applicationContext;
 
  public void printClass() {
    System.out.println("This is Class A: " + this);
    getClassB().printClass();
  }
 
  public ClassB getClassB() {
    return applicationContext.getBean(ClassB.class);
  }
 
  public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
    this.applicationContext = applicationContext;
  }
}
```

这样就能够在每次需要到Class B的时候手动去Context里找到新的bean。再跑一次测试后得到了以下输出：
```java
This is Class A: com.devhao.ClassA@4df828d7
This is Class B: com.devhao.ClassB@31206beb
This is Class A: com.devhao.ClassA@4df828d7
This is Class B: com.devhao.ClassB@3e77a1ed
This is Class A: com.devhao.ClassA@4df828d7
This is Class B: com.devhao.ClassB@3ffcd140
```
可以看到Class A的Hash Code在三次输出中保持不变，而Class B的却每次都不同，说明问题得到了解决，每次调用时用到的都是新的实例。

但是这样的写法就和Spring强耦合在一起了，Spring提供了另外一种方法来降低侵入性。

## Lookup方式

### @Lookup注解
Spring提供了一个名为@Lookup的注解，这是一个作用在方法上的注解，被其标注的方法会被重写，然后根据其返回值的类型，容器调用BeanFactory的getBean()方法来返回一个bean。
```java
@Component
public class ClassA {
  public void printClass() {
    System.out.println("This is Class A: " + this);
    getClassB().printClass();
  }
 
  @Lookup
  public ClassB getClassB() {
    return null;// 该方法会被cglib自动重写，所以直接返回null就可以。注意方法不能是private，否则cglib不能继承该方法
  }
}
```
可以发现简洁了很多，而且不再和Spring强耦合，再次运行测试依然可以得到正确的输出。

被标注的方法的返回值不再重要，因为容器会动态生成一个子类然后将这个被注解的方法重写/实现，最终调用的是子类的方法。

使用的@Lookup的方法需要符合如下的签名：
```java
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

### lookup-method(xml配置)
使用场景：一个singleton的bean使用一个prototype的bean时。

例子：
```java
// no more Spring imports! 
 
public abstract class CommandManager {
 
   public Object process(Object commandState) {
      // grab a new instance of the appropriate Command interface
      Command command = createCommand();
      // set the state on the (hopefully brand new) Command instance
      command.setState(commandState);
      return command.execute();
   }
 
    // okay... but where is the implementation of this method?
   protected abstract Command createCommand();
}

<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="command" class="fiona.apple.AsyncCommand" scope="prototype">
  <!-- inject dependencies here as required -->
</bean>
 
<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
  <lookup-method name="createCommand" bean="command"/>
</bean>
```

例子：
```java
public interface Bean2{//jiekou 
    Bean1 getBean1();//定义一个方法
}

<bean id="bean1"  class="..." scope="prototype" />
<bean id="bean2" class ="" >
    <lookup-method  name="getXXX" bean="car"/>
</bean>

这样配置后，就相当于如下的代码：
public class Bean2Impl implements Bean2,ApplicationContextAware{
    private ApplicationContext ctx;
    public Bean1 getBean1(){
        return (Bean1)ctx.getBean("bean1");
    }

    public void setApplicationContext(ApplicationContext ctx) throws BeanException{
        this.ctx=ctx;
    }

}
```



```java

```