
## jvisualvm下载地址

在高版本JDK（大于1.8或后期更新的1.8版本）中已经不会再自动集成 VisualVM，https://visualvm.github.io/index.html 到这里进行下载

下载的 visualvm 默认会找环境中的 jdk1.8，但如果你不是 jdk1.8 则可以修改配置（visualvm_218/etc/visualvm.conf），修改内容如下：
```
# 设置成自己安装的java路径
visualvm_jdkhome="D:\tool\java\java17"
```

## jvisualvm
`http://www.tianshouzhi.com/api/tutorials/jvm/352`

jvisualvm同jconsole都是一个基于图形化界面的、可以查看本地及远程的JAVA GUI监控工具，可以认为jvisualvm是jconsole的升级版，因此这里不再介绍jconsole，只介绍jvisualvm。jvisualvm是一个综合性的分析工具，可以认为其整合了jstack、jmap、jinfo等众多调试工具的功能，并以图形界面展示.

在JDK_HOME/bin目录下面，有一个jvisualvm.exe文件，双击即可打开，或者直接在命令行中输入"jvisualvm"也可。

主界面如下
![1487085026727079264](/assets/1487085026727079264.png)

侧边框介绍：  
本地：如果你本地有java进程启动了，那么在本地这个栏目就会显示。  
远程：就是监控的远程主机  
本地和远程展示的监控界面都是相同的  

jvisualvm分为四个选项卡：概述、监视、线程、抽样器，下面我们一一介绍

### “概述 ”选项卡
默认显示的就是概述选项卡，其中的信息相当于我们调用了jinfo命令获得，其还包含了两个子选项卡：

jvm参数栏：相当于我们调用`jinfo -flags <pid>`获得

系统属性栏：相当于我们调用`jinfo -sysprops <pid>`获得

![jvisualvm_01](/assets/jvisualvm_01.png)

### “监视”选项卡
主要显示了cpu、内存使用、类加载信息、线程信息等，这只是一个概要性的介绍，如下图：
![jvisualvm_02](/assets/jvisualvm_02.png)

右上角的"堆dump"是jmap命令的快捷方式，dump完成之后，可以直接点开。远程监控形式的话可以手工下载这个文件，通过"文件"->"装入"来进行分析，不过我们一般还是使用mat来进行分析，不会使用这个功能。

### “线程”选项卡
线程选项卡列出了所有线程的信息，并使用了不同的颜色标记，右下角的颜色表示了不同的状态。在这里可以查看所有线程，我提个建议，以后你在定义自己线程的时候给线程起一个名字，否则系统会用Thread-1、Thread-2以此类推来给创建的线程起名字。
![jvisualvm_03](/assets/jvisualvm_03.png)

右上角的线程dump会直接把线程信息dump到本地，相当于调用了jstack命令，如：
![jvisualvm_04](/assets/jvisualvm_04.png)

### “抽样器 ”选项卡
主要有"cpu"和"内存"两个按钮
![jvisualvm_05](/assets/jvisualvm_05.png)

点击cpu可以清楚的看到热点方法和各个线程所占cpu的情况：
![jvisualvm_06](/assets/jvisualvm_06.png)
![jvisualvm_07](/assets/jvisualvm_07.png)

点击内存可以清楚的看到各个类对象事例占的内存大小
![jvisualvm_08](/assets/jvisualvm_08.png)

### visual gc 插件安装
#### 离线安装visual gc
从visualvm的插件页面`https://visualvm.github.io/plugins.html`
到插件下载页面`https://visualvm.github.io/pluginscenters.html`
![axsaxsaxasx](/assets/axsaxsaxasx.png)
根据jdk的版本(1.7.0_79)选择插件，我的jdk应该右边第二个(右边的是java的visualvm即jvisualvm)。
![axasxasxddffgh](/assets/axasxasxddffgh.png)
点开是所有的插件，找到visual gc下载即可。

离线安装visual gc步骤：  
1、从主菜单中选择“工具”>“插件”。  
2、在“已下载”标签中，单击“添加插件”。  
3、打开选中提前下好的插件，逐步完成插件安装程序。  
4、可能需要重启

### visual gc
`https://www.cnblogs.com/reycg-blog/p/7805075.html`
`https://blog.csdn.net/weixin_34026276/article/details/85930598`

visual gc 是 visualvm 中的图形化查看 gc 状况的插件。
![1202311-20171108160915638-1259396529](/assets/1202311-20171108160915638-1259396529.png)

visual gc 工具分成三大块

* the Visual GC window（spaces）  
* the Graph window（graphs）  
* the Survivor Age Histogram window（histogram） 

#### spaces区域
我们看到上方图片中的 Spaces 就是 Visual GC window 了。它会分成 3 个竖直的部分，分别是 Perm 永生代，  Old 老年代和新生代。

新生代又分成 3 个部分 Eden 区， S0 survivor 区， S1 survivor 区.

每个方框中都使用不同的颜色表示，其中有颜色的区域是占用的空间，空白的部分是指剩余的空间。

当程序正在运行时，该部分区域就会动态显示，以直观的形式显示各个分区的动态情况。
![1202311-20171108165116763-11520902](/assets/1202311-20171108165116763-11520902.png)

#### Graphs区域
该区域包含多个以时间为横坐标的状态面板。

##### Compile Time
![1202311-20171109144835247-768276628](/assets/1202311-20171109144835247-768276628.png)
Compile Time(编译时间)：4609compiles 表示编译总数，9.115s表示编译累计时间。一个脉冲表示一次JIT编译，窄脉冲表示持续时间短，宽脉冲表示持续时间长。

编译时间表示虚拟机的 JIT 编译器编译热点代码的耗时。

Java 语言为了实现跨平台特性， Java 代码编译出来后形成的 class 文件中存储的是 byte code，jvm 通过解释的方式形成字节码命令，这种方式与 C/C++ 编译成二进制的方式　　相比要慢不少。

为了解决程序解释执行的速度问题， jvm 中内置了两个运行时编译器，如果一段 Java 代码被调用达到一定次数，就会判定这段代码为热点代码（hot spot code），并将这段代码交给 JIT 编译器编译成本地代码，从而提高运行速度。所以随着代码被编译的越来越彻底，运行速度应当是越来越快。

而 Java 运行器编译的最大缺点就是它进行编译时需要消耗程序正常的运行时间，也就是 compile time.

##### Class Loader Time
![1202311-20171109144719434-520181707](/assets/1202311-20171109144719434-520181707.png)
Class Loader Time(类加载时间): 21120loaded表示加载类数量, 0 unloaded表示卸载的类数量，28.669s表示类加载花费的时间

##### GC Time
![1202311-20171109144917278-2094290894](/assets/1202311-20171109144917278-2094290894.png)
22 collections 表示自监视以来一共经历了 22 次GC, 包括 Minor GC 和 Full GC   
2.030s 表示 gc 共花费了 2.030s  
Last Cause: Allocation Failure 表示上次发生 gc 的原因： 内存分配失败  

##### Eden Space
![1202311-20171109145332684-2073237401](/assets/1202311-20171109145332684-2073237401.png)
Eden Space (340.500M,185.000M): 91.012M   
表示 Eden Space 最大可分配空间  340.500M  
Eden Space 当前分配空间 185.000M  
Eden Space 当前占用空间 91.012M  
21 collections， 1.012s表示当前新生代发生 GC 的次数为 21 次, 共占用时间 1.012s    

##### Survivor 0 and Survivor 1
![1202311-20171109152427294-1771847281](/assets/1202311-20171109152427294-1771847281.png)
S0 和 S1 肯定有一个是空闲的，这样才能方便执行 minor GC 的操作，但是两者的最大分配空间是相同的。并且在 minor GC 时，会发生 S0 和S1 之间的切换。  
Survivor 1 (113.500M, 75.000M) : 36.590M   
表示 S1 最大分配空间 113.500M, 当前分配空间 75.000M, 已占用空间 36.590M   

##### Old Gen
![1202311-20171109152922716-1418649494](/assets/1202311-20171109152922716-1418649494.png)
Old Gen (682.500M, 506.500M) : 233.038M, 1 collections, 1.018s   
(682.500M, 506.500M) : 233.038M   
表示 OldGen 最大分配空间 682.500M， 当前空间  506.500M， 已占用空间 233.038M  
1 collections, 1.018s 表示老年代共发生了 1次 GC， 耗费了 1.018s 的时间。  

老年代 GC 也叫做 Full GC， 因为在老年代 GC 时总是会伴随着 Minor GC， 合起来就称为 Full GC。

##### Perm Gen
![1202311-20171109153928028-1978755818](/assets/1202311-20171109153928028-1978755818.png)
Perm Gen (256.000M, 227.500M) : 122.800M   
256.000M 表示最大可用空间，可以使用 -XX:MaxPermSize 指定永久代最大上限   
227.500M  表示当前永久代空间   
122.800M 表示永久代当前占用空间   

对 HotSpot 虚拟机来说，可以把永久代直接等同于方法区，其中会存储已经被jvm 加载的类信息，常量，静态变量，即时编译器编译后的代码等数据。

#### Histogram区域
![20191218 17103129222](/assets/20191218%2017103129222.png)
Tenuring Threshold：表示新生代年龄大于当前值则进入老年代

Max Tenuring Threshold：表示新生代最大年龄值。

Tenuring Threshold与Max Tenuring Threshold区别：Max Tenuring Threshold是一个最大限定，所有的新生代年龄都不能超过当前值，而Tenuring Threshold是个动态计算出来的临时值，一般情况与Max Tenuring Threshold相等，如果在Suivivor空间中，相同年龄所有对象大小的总和大于Survivor空间的一半，则年龄大于或者等于该年龄的对象就都可以直接进入老年代(如果计算出来年龄段是5，则Tenuring Threshold=5，age>=5的Suivivor对象都符合要求)，它才是新生代是否进入老年代判断的依据。

Desired Survivor Size：Survivor空间大小验证阙值(默认是survivor空间的一半)，用于Tenuring Threshold判断对象是否提前进入老年代。

Current Survivor Size：当前survivor空间大小

histogram柱状图：表示年龄段对象的存储柱状图

如果显示指定-XX:+UseParallelGC --新生代并行、老年代串行收集器 ，则histogram柱状图不支持当前收集器

### JMX 连接远程jvm
`-Djava.rmi.server.hostname`参数似乎加不加都行。

1、启动java进程时配置JMX参数

启动普通的jar程序
```
java -Dcom.sun.management.jmxremote.port=12345 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -jar foo.jar
```

tomcat 配置，修改远程tomcat的catalina.sh配置文件，在其中增加：
```
JAVA_OPTS="$JAVA_OPTS
-Djava.rmi.server.hostname=192.168.122.128
-Dcom.sun.management.jmxremote.port=18999
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false"
```

2、打开jvisualvm，点击远程或者右键远程，选择添加远程主机：
![159373-20160610120034199-1749896698](/assets/159373-20160610120034199-1749896698.png)

3、输入主机的名称，直接写ip，如下：
![159373-20160610120050058-357599247](/assets/159373-20160610120050058-357599247.png)

4、右键新建的主机，选择添加JMX连接，输入在ip:port即可。

5、双击打开。完毕！



