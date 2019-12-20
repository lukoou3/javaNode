## Btrace
BTrace是神器，每一个需要每天解决线上问题，但完全不用BTrace的Java工程师，都是可疑的。

BTrace的最大好处，是可以通过自己编写的脚本，获取应用的一切调用信息。而不需要重启应用！

只要定义脚本时不作大死(比如查看谁调用了HashMap的put方法)，直接在生产环境打开也没影响。

### BTrace简介
首先**BTrace就是为了解决线上问题而存在的**

BTrace是一种安全，动态的Java跟踪工具。BTrace通过动态（字节码）检测正在运行的Java程序的类来工作。BTrace将跟踪操作插入到正在运行的Java程序的类中，并对跟踪的程序类进行热交换。

限制：    
为了避免Btrace脚本的消耗过大影响真正业务，所以定义了一系列不允许的事情：比如不允许调用任何类的任何方法，只能调用BTraceUtils 里的一系列方法和脚本里定义的static方法。 比如不允许创建对象，比如不允许For 循环等等，更多规定看User Guide。

### 使用场景
1.查看某一个方法中入参
 
2.查看某一个方法的响应时间
 
3.查看某一个方法中所有外部调用的响应时间，方便定位方法响应慢的具体位置及原因
 
4.查看谁调用了 System.gc(),及其对应的调用栈


### BTrace限制
BTrace植入过的代码，会一直在，直到应用重启为止。

为了保证跟踪操作是“只读”（即跟踪操作不会改变跟踪的程序状态）和有界（即跟踪操作在有限时间内终止），BTrace程序只允许执行一组有限的行动。特别是BTrace类

* 不能创建新的对象。    
* 不能创造新的阵列。    
* 不能抛出异常。    
* 不能捕捉异常。    
* 不能让任意实例或静态方法调用-只有public static方法的com.sun.btrace.BTraceUtils 类可以从BTrace程序调用。    
* 不能分配到目标程序的类和对象的静态或实例字段。但是，BTrace类可以分配给它自己的静态字段（“跟踪状态”可以变异）。    
* 不能有实例字段和方法。static public voidBTrace类只允许返回方法。所有领域都必须是静态的。    
* 不可以具有外，内，嵌套的或局部类。    
* 不可以具有同步块或同步方法。    
* 不能有环（for, while, do..while）    
* 不可以延伸任意类（超类必须java.lang.Object中）    
* 不可以实现接口。    
* 不可以包含断言语句。    
* 不可以使用类文字。  

### 下载
`http://github.com/btraceio/btrace`，在Release页面里下载最新版本，解压就可以使用。zip那个是Windows版本的。

配置环境变量：   
新建环境变量BTRACE_HOME值为`E:/btrace-bin-1.3.9`，然后编辑Path变量，在值的末尾追加;%BTRACE_HOME%/bin即可，验证是否安装成功，打开cmd，输入btrace测试是否生效。

### 使用步骤
BTrace使用非常简单，只需要编写的脚本文件即可，不用编译，也不需要引入其他的jar(AnyType相当于scala里的Any可以表示所有类型，要引入具体的类的话，相当麻烦，不如不引)。

编写完脚本后只需要运行以下命令即可，pid为java程序的进程id：
```sh
#用这个就行了
btrace <pid> BtraceScript.java

#输出到文件
btrace <pid> BtraceScript.java > btracelog.log
```

#### 官网中运行BTrace的步骤
* 查找要跟踪的目标Java进程的进程ID。您可以使用jps工具查找pid。    
* 编写BTrace程序 - 您可能想要开始修改其中一个 示例。    
* 通过以下命令行运行btrace工具：    
```
btrace <pid> <btrace-script>
```

#### 编写脚本时代码提示
其实编写这个脚本不需要引入多少类，也不需要编译，使用一般的编辑工具即可。每次复制一个模板改改参数就行了。

一个脚本事例：

监控的方法签名：
```java
List<Object> termInfoPageList(DisplayParam displayParam,DacTermInfoParam param, boolean gradeHasSlave)
```
```java
@OnMethod(
        clazz = "com.dp.dac.dao.DacMonitorOverviewDao",
        method = "termInfoPageList",
        location = @Location(Kind.ENTRY)
)
//AnyType表示所有类型而省去引入额外的类；参数个数和类型要完全对应，可以区分函数的重载；
public static void methodEntry2(@ProbeClassName String pcn, @ProbeMethodName String pmn, AnyType displayParam, AnyType param, boolean gradeHasSlave) {
    //print all fields
    BTraceUtils.printFields(displayParam);
    BTraceUtils.printFields(param);
    //print one field
    Field filed = BTraceUtils.field("com.dp.plat.param.DisplayParam", "currentpage");
    BTraceUtils.println(BTraceUtils.get(filed, displayParam));
    BTraceUtils.println(pcn + "," + pmn);
    BTraceUtils.println();
}
```

下面说说用Maven项目怎么编写BTrace脚本吧。

如果是在Maven项目中开发，那么首先需要引入BTrace的Jar包，由于Maven的中央仓库中只有1.x版本的BTrace，并没有高版本的，所以一般的做法是自己编译BTrace源码，将高版本的Jar发布到私服（Nexus）中，为简单起见，此处通过Maven指定依赖本地Jar即可，修改pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.btrace</groupId>
    <artifactId>BtraceTest</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--btrace start-->
        <dependency>
            <groupId>com.sun.tools.btrace</groupId>
            <artifactId>btrace-agent</artifactId>
            <version>1.3.11.3</version>
            <scope>system</scope>
            <systemPath>D:/UMCv5Dev/UMC/btrace-bin-1.3.11.3/build/btrace-agent.jar</systemPath>
        </dependency>
        <dependency>
            <groupId>com.sun.tools.btrace</groupId>
            <artifactId>btrace-boot</artifactId>
            <version>1.3.11.3</version>
            <scope>system</scope>
            <systemPath>D:/UMCv5Dev/UMC/btrace-bin-1.3.11.3/build/btrace-boot.jar</systemPath>
        </dependency>
        <dependency>
            <groupId>com.sun.tools.btrace</groupId>
            <artifactId>btrace-client</artifactId>
            <version>1.3.11.3</version>
            <scope>system</scope>
            <systemPath>D:/UMCv5Dev/UMC/btrace-bin-1.3.11.3/build/btrace-client.jar</systemPath>
        </dependency>
        <!--btrace end-->
    </dependencies>

</project>
```

### 一个简单的BTrace程序
摘抄自官网：
```java
// import all BTrace annotations
import com.sun.btrace.annotations.*;
// import statics from BTraceUtils class
import static com.sun.btrace.BTraceUtils.*;

// @BTrace annotation tells that this is a BTrace program
@BTrace
public class HelloWorld {
 
    // @OnMethod annotation tells where to probe.
    // In this example, we are interested in entry 
    // into the Thread.start() method. 
    @OnMethod(
        clazz="java.lang.Thread",
        method="start"
    )
    public static void func() {
        // println is defined in BTraceUtils
        // you can only call the static methods of BTraceUtils
        println("about to start a thread!");
    }
}
```

上面的BTrace程序可以针对正在运行的Java进程运行。每当目标程序即将通过Thread.start()方法启动一个线程时，在BTrace客户端将打印“about to start a thread!”。

启动BTrace程序很简单(12345为监控java进程的pid)：
```sh
btrace 12345 HelloWorld.java
```

### BTrace 事例
#### 拦截方法定义
拦截方法定义,也说是 `@OnMethod` 注解的作用

Btrace使用@OnMethod注解定义需要分析的方法入口，在@OnMethod注解中，需要指定class、method以及location等，class表明需要监控的类，method表明需要监控的方法，指定方式如下：

##### 精准定位(使用全限定名)
就是HelloWorld的例子，精确定义要监控的类与方法。如：
```java
clazz="com.metty.rpc.common.BtraceCase", method="add"
```

##### 正则表达式定位
可以用正则表达式，批量定义需要监控的类与方法。正则表达式需要写在两个 "/" 中间。

下例监控javax.swing下的所有类的所有方法....可能会非常慢，建议范围还是窄些。
```java
@OnMethod(clazz="/javax\\.swing\\..*/", method="/.*/")  
public static void swingMethods( @ProbeClassName String probeClass, @ProbeMethodName String probeMethod) {  
   print("entered " + probeClass + "."  + probeMethod);  
} 
```
通过在拦截函数的定义里注入@ProbeClassName String probeClass, @ProbeMethodName String probeMethod 参数，告诉脚本实际匹配到的类和方法名。

##### 按接口，父类，Annotation定位
比如我想匹配所有的Filter类，在接口或基类的名称前面，加个 `+` 就行
```java
@OnMethod(clazz="+com.vip.demo.Filter", method="doFilter")
```

也可以按类或方法上的annotaiton匹配，也就是注解匹配，前面加上@就行
```java
@OnMethod(clazz="@javax.jws.WebService", method="@javax.jws.WebMethod")
```

##### 构造函数
构造函数的名字是 `<init>`，如果需要分析构造方法，需要指定method="`<init>`"
```java
@OnMethod(clazz="java.net.ServerSocket", method="<init>")
```

##### 静态内部类
静态内部类的写法，是在类与内部类之间加上"`$`"
```java
@OnMethod(clazz="com.vip.MyServer$MyInnerClass", method="hello")
```

##### 函数重载
如果有多个同名的函数，想区分开来，可以在拦截函数上定义不同的参数列表。

其实不只是区分函数重载的时候需要明确区分参数个数和类型，任何的方法拦截只要定义了参数，就必须匹配得上参数类型，不然没用。

#### 拦截时机
拦截时机， 也就@Location注解的作用。不写Location，默认就是刚进入函数的时候(Kind.ENTRY)。

定义Btrace对方法的拦截位置，通过@Location注解指定，默认为Kind.ENTRY。可以为同一个函数的不同的Location，分别定义多个拦截函数。

##### Kind.Entry与Kind.Return
* Kind.ENTRY：在进入方法时，调用Btrace脚本    
* Kind.RETURN：方法执行完时，调用Btrace脚本，只有把拦截位置定义为Kind.RETURN，才能获取方法的返回结果@Return和执行时间  

不写Location，默认就是刚进入函数的时候(Kind.ENTRY)。
但如果你想获得函数的返回结果或执行时间，则必须把切入点定在返回(Kind.RETURN)时。
```java
OnMethod(clazz = "java.net.ServerSocket", method = "getLocalPort", location = @Location(Kind.RETURN))  
public static void onGetPort(@Return int port, @Duration long duration)  
```

duration的单位是纳秒，要除以 1,000,000 才是毫秒。

##### Kind.Error, Kind.Throw和 Kind.Catch
异常抛出(Throw)，异常被捕获(Catch)，异常没被捕获被抛出函数之外(Error)，主要用于对某些异常情况的跟踪。
在拦截函数的参数定义里注入一个`@TargetInstance Throwable exception`的参数，代表异常。
```java
@OnMethod(clazz = "java.net.ServerSocket", method = "bind", location = @Location(Kind.ERROR))  
public static void onBind(@TargetInstance Throwable exception)  
```

根据我的实际测试发现，Catch能捕捉到所有被Catch住的异常，其它的异常都能被Error捕捉到，而Throw似乎只能捕获到被抛出的非runtime异常，对于runtime类型的异常即使方法里标明throw也捕获不到(其实就不用throw，编译器也不提示报错，而非runtime异常就必须throw，当然是在没有catch的情况下)。

##### Kind.Call
Kind.CALL：分析方法中调用其它方法的执行情况，比如在execute方法中，想获取add方法的是否调用。想获取add方法的执行耗时，必须把where设置成Where.AFTER.

下例定义监控bind()函数里调用的所有其他函数:
```java
@OnMethod(clazz = "java.net.ServerSocket", method = "bind", location = @Location(value = Kind.CALL, clazz = "/.*/", method = "/.*/", where = Where.AFTER))  
public static void onBind(@Self Object self, @TargetInstance Object instance, @TargetMethodOrField String method, @Duration long duration)  
```
所调用的类及方法名所注入到`@TargetInstance`与 `@TargetMethodOrField`中。

​静态函数中，instance的值为空。如果想获得执行时间，必须把Where定义成AFTER。

注意这里，一定不要像下面这样大范围的匹配，否则这性能就算是神仙也没法救了：
```java
@OnMethod(clazz = "/javax\\.swing\\..*/", method = "/.*/", location = @Location(value = Kind.CALL, clazz = "/.*/", method = "/.*/"))
```

##### Kind.Line
Kind.LINE：通过设置line，可以监控代码是否执行到类指定的位置。**Kind.LINE监控的是类，而不是方法。**

下例监控代码是否到达了Socket类的第363行。
```java
@OnMethod(clazz = "java.net.ServerSocket", location = @Location(value = Kind.LINE, line = 363))  
public static void onBind4() {  
   println("socket bind reach line:363");  
}  
```

line还可以为-1，然后每行都会打印出来，加参数int line 获得的当前行数。此时会显示函数里完整的执行路径，但肯定又非常慢。
```java
@BTrace
public class AllLines {
    @OnMethod(
        clazz="java.lang.Thread",
        location=@Location(value=Kind.LINE, line=-1)
    )
    public static void online(@ProbeClassName String pcn, @ProbeMethodName String pmn, int line) {
        print(pcn + "." + pmn +  ":" + line);
    }
}
```

#### 打印this，参数 与 返回值
```java
@OnMethod(clazz = "java.io.File", method = "createTempFile", location = @Location(value = Kind.RETURN))  
public static void o(@Self Object self, String prefix, String suffix, @Return AnyType result) 
```
如果想打印它们，首先按顺序定义用@Self 注释的this， 完整的参数列表，以及用@Return 注释的返回值。
需要打印哪个就定义哪个，**不需要的可以不要定义。但定义一定要按顺序**，比如参数列表不能跑到返回值的后面。

Self：    
* 如果是静态函数， self为空。    
* 如果使用了非JDK的类，命令行里要指定classpath。不过，因为BTrace里不允许调用类的方法，所以定义具体类很多时候也没意思，所以self定义为Object就够了。   

参数：    
* 参数数列表要么不要定义，要定义就要定义完整，否则BTrace无法处理不同参数的同名函数。    
* 如果有些参数你实在不想引入非JDK类，又不会造成同名函数不可区分，可以用AnyType来定义（不能用Object）。    
* 如果拦截点用正则表达式中匹配了多个函数，函数之间的参数个数不一样，你又还是想把参数打印出来时，可以用AnyType[] args来定义。    
* 但不知道是不是当前版本的bug，AnyType[] args 不能和 location＝Kind.RETURN 同用，否则会进入一种奇怪的静默状态，只要有一个函数定义错了，整个Btrace就什么都打印不出来。  

结果：    
* 同理，结果也可以用AnyType来定义，特别是用正则表达式匹配多个函数的时候，连void都可以表示。   

#### 查看对象的实例属性值
再次强调，为了保证性能不受影响，Btrace不允许调用任何实例方法。    
比如不能调用getter方法（怕在getter里有复杂的计算），只会通过直接反射来读取属性名。    
又比如，除了JDK类，其他类toString时只会打印其类名＋System.IdentityHashCode。    
println, printArray，都按上面的规律进行，所以只能打打基本类型。    
如果想打印一个Object的属性，用printFields()来反射。    
如果只想反射某个属性，参照下面打印Port属性的写法。**从性能考虑，应把field用静态变量缓存起来**。    

```java
import java.lang.reflect.Field;  
//JDK的类这样写就行  
private static Field fdFiled = field("java.io,FileInputStream", "fd");  
//非JDK的类，要给出ClassLoader，否则ClassNotFound  
private static Field portField = field(classForName("com.vip.demo.MyObject", contextClassLoader()), "port");  
public static void onChannelRead(@Self Object self) {  
    println("port:" + getInt(portField, self));  
}  
```

这样也是可以的(我没试过上面那种，感觉用下面这个就行了)：
```java
@BTrace  
public class HelloWorld {  
    @OnMethod(clazz = "java.io.File", method = "createTempFile", location = @Location(value = Kind.RETURN))  
    public static void o(@Self Object self, TmpTest param, @Return AnyType result){  
        Field field = BTraceUtils.filed("com.play.zhazah.TmpTest","data");  
        println("data = " + BTraceUtils.get(field, param));  
    }  
}  
```

#### TLS，拦截函数间的通信机制
如果要多个拦截函数之间要通信，可以使用@TLS定义 ThreadLocal的变量来共享
```java
@TLS  
private static int port = -1;  
@OnMethod(clazz = "java.net.ServerSocket", method = "<init>")  
public static void onServerSocket(int p){  
    port = p;  
}  
@OnMethod(clazz = "java.net.ServerSocket", method = "bind")  
public static void onBind(){  
  println("server socket at " + port);  
}  
```

#### 打印慢调用
下例打印所有用时超过1毫秒的filter。
```java
@OnMethod(clazz = "+com.vip.demo.Filter", method = "doFilter", location = @Location(Kind.RETURN))  
public static void onDoFilter2(@ProbeClassName String pcn,  @Duration long duration) {  
    if (duration > 1000000) {  
        println(pcn + ",duration:" + (duration / 100000));  
    }  
}  
```
最好能抽取了打印耗时的函数，减少代码重复度。

定位到某一个Filter慢了之后，可以直接用Location(Kind.CALL)，进一步找出它里面的哪一步慢了。

#### 谁调用了这个函数
比如，谁调用了System.gc() ?
```java
@OnMethod(clazz = "java.lang.System", method = "gc")  
public static void onSystemGC() {  
    println("entered System.gc()");  
    jstack(); //打印堆栈信息  
}  
```

通过查看调用栈，可以很清楚的发现哪个类哪个方法调用了System.gc()

#### 统计方法的调用次数，且每隔1分钟打印调用次数
```java
@Export static AtomicLong counter = new AtomicLong();  
@OnMethod(class="com.**.MyObject",method="add")  
public static void run(){  
    counter.getAndIncrement();  
}  
@OnTimer(1000*60)  
public static void run(){  
    BTraceUtils.println("count: " + connter.get());  
    counter.set(0);  
}  
```
Btrace的@OnTimer注解可以实现定时执行脚本中的一个方法

#### 检查死锁
官方的事例：
```java
import com.sun.btrace.annotations.*;
import static com.sun.btrace.BTraceUtils.Threads.*;

/**
 * This BTrace program demonstrates deadlocks
 * built-in function. This example prints 
 * deadlocks (if any) once every 4 seconds.
 */ 
@BTrace public class Deadlock {
    @OnTimer(4000)
    public static void print() {
        deadlocks();        
    }
}
```

上面代码就是每4秒钟检查以下死锁，dealdlocks方法是BTrace工具提供的静态方法

#### 其他事例
在Btrace的解压目录中，samples文件夹有很多事例。自己去看。

### BTrace api

#### @OnMethod注解
@OnMethod注解常用的属性：

* "clazz"属性:用来指定目标类名    
* "method"属性:用来指定被trace的方法    
* "location"属性:用来指定拦截时机    
* "type"属性:用来指定方法签名(重载的时候，只有方法名不行)如：type="int (int, int)"  

location后面单独讲，先说"clazz"和"method"

使用全限定名
```java
clazz="cn.freemethod.btra.BtraceMap"
method="sayHello"
```
注意：静态内部类的写法，是在类与内部类之间加上"$"

使用正则表达式 方法和类都可以使用正则表达式
```java
clazz="/java\\.lang\\..*/"
method="/.*/"
```

使用接口或者基类 接口在前面添加"+"表明是一个接口就可了
```java
clazz="+xxx.xxx.Interface"
```

使用注解 类和方法都可以使用注解，在前面加"@"就可以了
```java
clazz="@xxx.xxx.Annotation"
method="@xxx.xxx.Annotation"
```

构造方法 构造方法指定method="`<init>`"就可以了

注意：正则表达式需要写在两个 "/",正则表达式的范围要尽可能的小，不然会非常慢

注意：@OnMethod都是注解的public static void方法

#### @Location注解
定义Btrace对方法的拦截位置 通过@Location注解指定 默认为Kind.ENTRY

* Kind.ENTRY 在进入方法时，调用Btrace脚本    
* Kind.RETURN 方法执行完时，调用Btrace脚本，只有把拦截位置定义为Kind.RETURN，才能获取方法的返回结果@Return和执行时间@Duration    
* Kind.CALL 分析方法中调用其它方法的执行情况    
* Kind.LINE 通过设置line，可以监控代码是否执行到指定的位置    
* Kind.ERROR 异常未捕获被抛出方法之外    
* Kind.THROW 异常抛出    
* Kind.CATCH 异常被捕获

#### 参数注解
* @Self用来指定被trace方法的this    
* @Return用来指定被trace方法的返回值    
* @ProbeClassName用来指定被trace的类名    
* @ProbeMethodName用来指定被trace的方法名    
* @TargetInstance用来指定被trace方法内部被调用到的实例    
* @TargetMethodOrField用来指定被trace方法内部被调用的方法名

#### @OnTimer注解
定时触发Trace，时间可以指定，单位为毫秒

#### @TLS
定义ThreadLocal的共享变量

#### @OnError注解
当trace代码抛异常或者错误时，该注解的方法会被执行.如果同一个trace脚本中其他方法抛异常,该注解方法也会被执行

#### @OnExit
当trace方法调用内置exit(int)方法(用来结束整个trace程序),该注解的方法会被执行

#### BTraceUtils方法
```java
import static com.sun.btrace.BTraceUtils.exit;  
import static com.sun.btrace.BTraceUtils.field;  
import static com.sun.btrace.BTraceUtils.get;  
import static com.sun.btrace.BTraceUtils.jstack;  
import static com.sun.btrace.BTraceUtils.printEnv;  
import static com.sun.btrace.BTraceUtils.printFields;  
import static com.sun.btrace.BTraceUtils.printProperties;  
import static com.sun.btrace.BTraceUtils.printVmArguments;  
import static com.sun.btrace.BTraceUtils.println;  

import com.sun.btrace.BTraceUtils.Strings;  
import com.sun.btrace.BTraceUtils.Sys;  
import com.sun.btrace.annotations.BTrace;  
import com.sun.btrace.annotations.Duration;  
import com.sun.btrace.annotations.Kind;  
import com.sun.btrace.annotations.Location;  
import com.sun.btrace.annotations.OnMethod;  
import com.sun.btrace.annotations.Self;
```

获取当前线程名称
```java
BTraceUtils.Threads.name(BTraceUtils.currentThread())
```

获取Hash code
```java
BTraceUtils.identityHashCode()
```

获取对象的类名称
```java
BTraceUtils.Reflective.name(clazz)
```

返回类
```java
BTraceUtils.classOf(obj)
```

打印信息
```java
BTraceUtils.print()
BTraceUtils.println()
BTraceUtils.printArray()
printVmArguments()
printProperties()
printEnv()
printFields(obj)
```

获取值
```java
field("java.lang.Thread", "name")
field(classForName("xxx.xxx.ClassName", contextClassLoader()),"fieldName")
get(field, obj)
getBoolean()
getInt()
```

系统相关
```java
exit()//退出BTrace
heapUsage()//堆使用情况
nonHeapUsage()//非堆使用情况
jstack()//堆栈信息
Sys.Memory.dumpHeap("data.bin")//堆dump
```

### Btrace 总结
#### btrace的使用是否会对java进程造成影响？
btrace的使用是否会对java进程造成影响？（影响是肯定的，不过影响不大）

装载时的影响：
btrace每次使用，都会重新load所有的class。当然如果OnMethod不匹配，是不会被重新装载。所以跟你的OnMethod的匹配规则很有关系，如果使用+java.lang.Object。那就死定了。

退出后的影响：
btrace监控每次退出后，原先所有的class都不会被恢复，你的所有的监控代码依然一直在运行 

再看一下BTraceRuntime中对应方法的实现：
```java
private volatile boolean disabled;  
  
public static boolean enter(BTraceRuntime current)  
{  
    if (current.disabled) return false;  
    return map.enter(current);  
}  
```
每次执行你的监控代码之前会先进行一个判断，判断当前是否处于监控中。你的客户端发起了exit指令后，该方法判断false，直接return。
所以btrace使用退出后会让你的代码多走了一个方法调用+一个对象属性判断，所以说影响还是非常的少

#### OnMethod的Location使用
说明： self, ProbeClassName , ProbeMethodName 在任何的Kind中都支持，所以就不在每个表格中赘述。

| Kind       | Where.BEFORE                                            | Where.AFTER                                                      |
| ---------- | ------------------------------------------------------- | ---------------------------------------------------------------- |
| ARRAY_GET  | 数组长度(int) , 数组类型(type)                          | @return , 数组长度(int) , 数组类型(type)                         |
| ARRAY_SET  | 原始数组类型(type) , 数组长度(int) , 目标数组类型(type) | @return，原始数组类型(type) , 数组长度(int) , 目标数组类型(type) |
| CALL       | 方法参数 , @TargetInstance , @TargetMethodOrField       | 方法参数, @return , @TargetInstance , @TargetMethodOrField       |
| CATCH      | 异常类型(type)                                          | 异常类型(type)                                                   |
| CHECKCAST  | 转型的目标类型                                          | 转型的目标类型                                                   |
| ENTRY      | 方法参数                                                | 方法参数                                                         |
| ERROR      | 异常类型(throwable type)                                | 异常类型(throwable type)                                         |
| FIELD_GET  | @TargetInstance,@TargetMethodOrField                    | @TargetInstance,@TargetMethodOrField,@return                     |
| FIELD_SET  | fldValueIndex,@TargetInstance,@TargetMethodOrField      | fldValueIndex,@TargetInstance,@TargetMethodOrField               |
| INSTANCEOF | 转型的目标类型                                          | 转型的目标类型                                                   |
| LINE       | 行数                                                    | 行数                                                             |
| NEW        | 对象类名                                                | @return                                                          |
| NEWARRAY   | 数组内部对象类名，类名                                  | 数组内部对象类名，类名, @return                                  |
| RETURN     | 无                                                      | 参数，@return , @Duration                                        |
| SYNC_ENTRY | sync对象                                                | sync对象                                                         |
| SYNC_EXIT  | sync对象                                                | sync对象                                                         |
| THROW      | 异常类型                                                | 异常类型                                                         |

### 实战

#### 我的测试代码
监控的函数的签名
```java
//DacMonitorOverviewDao
List<DacGradeQueryResult> getDeviceScanProgressList()
List<DacMonitorOverviewLogData> getLogList(boolean gradeHasSlave)
List<Object> termInfoPageList(DisplayParam displayParam,DacTermInfoParam param, boolean gradeHasSlave)

//DacMonitorOverviewAction
String execute()

//BaseAction
DisplayParam getDisplayParam()

//DacTerminalInfoDao
String errorTest()
String errorTest2() throws Exception
String errorTest3()
```

BTrace脚本：
```java
package com;

import com.sun.btrace.AnyType;
import com.sun.btrace.BTraceUtils;
import com.sun.btrace.annotations.*;

import java.lang.reflect.Field;

@BTrace
public class BtraceTest {
	@OnMethod(
			clazz = "com.dp.dac.dao.DacMonitorOverviewDao",
			method = "getDeviceScanProgressList"
	)
	//不写Location，默认就是刚进入函数的时候(Kind.ENTRY)
	public static void func() {
		BTraceUtils.println("getDeviceScanProgressList called");
		BTraceUtils.println();
	}

	@OnMethod(
			clazz = "com.dp.dac.dao.DacMonitorOverviewDao",
			method = "getLogList",
			location = @Location(Kind.ENTRY)
	)
	public static void methodEntry(@ProbeClassName String pcn, @ProbeMethodName String pmn, AnyType[] args) {
		BTraceUtils.printArray(args);
		BTraceUtils.println(pcn + "," + pmn);
		BTraceUtils.println();
	}

	@OnMethod(
			clazz = "com.dp.dac.dao.DacMonitorOverviewDao",
			method = "termInfoPageList",
			location = @Location(Kind.ENTRY)
	)
	//AnyType表示所有类型而省去引入额外的类；参数个数和类型要完全对应，可以区分函数的重载；
	public static void methodEntry2(@ProbeClassName String pcn, @ProbeMethodName String pmn, AnyType displayParam, AnyType param, boolean gradeHasSlave) {
		//print all fields
		BTraceUtils.printFields(displayParam);
		BTraceUtils.printFields(param);
		//print one field
		Field filed = BTraceUtils.field("com.dp.plat.param.DisplayParam", "currentpage");
		BTraceUtils.println(BTraceUtils.get(filed, displayParam));
		BTraceUtils.println(pcn + "," + pmn);
		BTraceUtils.println();
	}

	@OnMethod(
			clazz = "com.dp.dac.action.DacMonitorOverviewAction",
			method = "execute",
			location = @Location(Kind.RETURN)
	)
	//结果也可以用AnyType来定义, 特别是用正则表达式匹配多个函数的时候，连void都可以表示；
	public static void methodReturn(@ProbeClassName String pcn, @ProbeMethodName String pmn, @Return AnyType result) {
		BTraceUtils.println(pcn + "," + pmn + "," + result);
		BTraceUtils.println();
	}

	@OnMethod(
			clazz = "+com.dp.plat.util.BaseAction",
			method = "getDisplayParam",
			location = @Location(Kind.RETURN)
	)
	//匹配所有的Action类，在接口或基类的名称前面，加个+ 就行
	public static void methodReturn2(@ProbeClassName String pcn, @ProbeMethodName String pmn, @Return AnyType result) {
		BTraceUtils.println(pcn + "," + pmn);
		Field filed = BTraceUtils.field("com.dp.plat.param.DisplayParam", "currentpage");
		BTraceUtils.println(BTraceUtils.get(filed, result));
		BTraceUtils.println();
	}

	@OnMethod(
			clazz="com.dp.plat.type.DateTime",
			method="<init>"
	)
	//匹配构造函数的调用
	public static void anyRead(@ProbeClassName String pcn, @ProbeMethodName String pmn, AnyType[] args) {
		BTraceUtils.println(pcn+","+pmn);
		BTraceUtils.printArray(args);
		BTraceUtils.println();
	}

	@OnMethod(
			clazz = "com.dp.dac.dao.DacMonitorOverviewDao",
			method = "getLogList",
			location = @Location(value = Kind.CALL, clazz = "com.dp.dac.data.DacMonitorOverviewLogData", method = "setTimeLong")
	)
	//DacMonitorOverviewDao类getLogList方法中DacMonitorOverviewLogData类setTimeLong方法的调用情况
	public static void methodCall(@TargetInstance Object instance, @TargetMethodOrField String method, AnyType[] args) {
		BTraceUtils.println(instance);
		BTraceUtils.println(method);
		BTraceUtils.printArray(args);
		BTraceUtils.println();
	}

	@OnMethod(
			clazz = "com.dp.dac.dao.DacMonitorOverviewDao",
			method = "getLogList",
			location = @Location(value = Kind.CALL, clazz = "java.text.SimpleDateFormat", method = "format", where = Where.AFTER)
	)
	//DacMonitorOverviewDao类getLogList方法中DacMonitorOverviewLogData类setTimeLong方法的调用情况
	public static void methodCall2(@TargetInstance Object instance, @TargetMethodOrField String method, AnyType[] args, @Return AnyType result) {
		BTraceUtils.println(instance);
		BTraceUtils.println(method);
		BTraceUtils.printArray(args);
		BTraceUtils.println(result);
		BTraceUtils.println();
	}

	@OnMethod(
			clazz = "com.dp.dac.dao.DacMonitorOverviewDao",
			location = @Location(value = Kind.LINE, line = 1516)
	)
	//监控代码是否到了某个类的第几行，line还可以为-1，然后每行都会打印出来，加参数int line 获得的当前行数。此时会显示函数里完整的执行路径，但肯定又非常慢。
	public static void online(@ProbeClassName String pcn, @ProbeMethodName String pmn, int line) {
		BTraceUtils.println(pcn + "." + pmn + ":" + line);
	}

	@OnMethod(
			clazz = "com.dp.dac.dao.DacMonitorOverviewDao",
			location = @Location(value = Kind.LINE, line = 1518)
	)
	public static void online2(@ProbeClassName String pcn, @ProbeMethodName String pmn, int line) {
		BTraceUtils.println(pcn + "." + pmn + ":" + line);
	}

	@OnMethod(
			clazz = "com.dp.dac.dao.DacMonitorOverviewDao",
			location = @Location(value = Kind.LINE, line = 1552)
	)
	public static void online3(@ProbeClassName String pcn, @ProbeMethodName String pmn, int line) {
		BTraceUtils.println(pcn + "." + pmn + ":" + line);
	}

	@OnMethod(clazz = "com.dp.dac.dao.DacMonitorOverviewDao",
			method = "/.*/",
			location = @Location(Kind.RETURN))
	public static void duration(@ProbeMethodName String pmn, @Duration long duration) {
		if (duration >= 100000) {
			BTraceUtils.println(pmn + ",duration:" + (duration / 100000) + " ms");
		}
	}

	@OnMethod(
			clazz = "com.dp.dac.dao.DacTerminalInfoDao",
			method = "/errorTest[23]?/",
			location = @Location(Kind.ERROR)
	)
	//这个似乎能捕捉到RuntimeException和throws Exception，范围比Kind.THROW要广
	//获取异常要用@TargetInstance注解
	public static void methodError(@ProbeClassName String pcn, @ProbeMethodName String pmn,@TargetInstance Throwable exception) {
		BTraceUtils.println("ERROR");
		BTraceUtils.println(pcn + "." + pmn);
		BTraceUtils.println(exception);
		//BTraceUtils.Threads.jstack(exception);
		BTraceUtils.println();
	}

	@OnMethod(
			clazz = "com.dp.dac.dao.DacTerminalInfoDao",
			method = "/errorTest[23]?/",
			location = @Location(Kind.THROW)
	)
	//这个似乎只能捕捉到抛出的非RuntimeException异常
	public static void methodThrow(@ProbeClassName String pcn, @ProbeMethodName String pmn,@TargetInstance Throwable exception) {
		BTraceUtils.println("THROW");
		BTraceUtils.println(pcn + "." + pmn);
		BTraceUtils.println(exception);
		//BTraceUtils.Threads.jstack(exception);
		BTraceUtils.println();
	}

	@OnMethod(
			clazz = "com.dp.dac.dao.DacTerminalInfoDao",
			method = "/errorTest[23]?/",
			location = @Location(Kind.CATCH)
	)
	//方法里面所有被CATCH的异常都可以捕捉到
	public static void methodCatch(@ProbeClassName String pcn, @ProbeMethodName String pmn,@TargetInstance Throwable exception) {
		BTraceUtils.println("CATCH");
		BTraceUtils.println(pcn + "." + pmn);
		BTraceUtils.println(exception);
		//BTraceUtils.Threads.jstack(exception);
		BTraceUtils.println();
	}

}
```
监控的异常函数的具体定义
```java
public String errorTest()
{
    String str = "1a1";
    
    int num = Integer.parseInt(str);
    
    return num + "";
}

public String errorTest2() throws Exception
{	
    throw new Exception("error");
}

public String errorTest3()
{	
    String str = "1a1";
    int num = 0;
    
    try
    {
        num = Integer.parseInt(str);
    }
    catch (NumberFormatException e)
    {
        //e.printStackTrace();
    }
    
    str = "2a2";
    
    try
    {
        num = Integer.parseInt(str);
    }
    catch (NumberFormatException e)
    {
        //e.printStackTrace();
    }
    
    return num + "";
}
```



```java

```

