## JRE、JDK和JVM之间的关系
Java学了有几年了，要让详细说说JRE、JDK、JVM，确实不能脱口而出。下面在别人博客的基础上，总结一下。

* **JVM（Java Virtual Machine 即Java虚拟机）**

它是整个Java实现跨平台的核心部分。所有的Java程序会首先被编译为.class的类文件，这种类文件可以在虚拟机执行，也就是说class并不直接与机器的操作系统相对应，而是经过虚拟机间接与操作系统交互，由虚拟机将程序解释给本地系统执行。

也可以理解为是一个虚拟出来的计算机，具备着计算机的基本运算方式，它主要负责将Java程序生成的字节码文件解释成具体系统平台上的机器指令。让具体的平台如Windows/Linux运行这些Java程序。

* **JRE（Java Runtime Environment 即Java运行环境）**

光有JVM还不能执行class文件，因为在解释class的时候JVM需要调用解释所需要的类库lib。在JDK的安装目录里可以看到jre目录，里面有两个文件夹bin和lib，在这里可以认为bin里的就是JVM，lib中则是jvm工作所需要的类库，而jvm和lib和起来就称为jre。所以，在写完Java程序编译成.class之后，可以把.class文件和jre一起打包发给别人，这样别人就可以运行你的程序了。如果想要运行一个开发好的Java程序，只需要给计算机安装JRE即可。

* **JDK（Java Development Kit 即Java开发工具包）**

是程序开发者用来来编译、调试java程序用的开发工具包。JDK的工具也是Java程序，也需要JRE才能运行。

在jdk的安装目录下有6个文件夹、一个src类库源码压缩包和其他几个声明文件。其中，真正在运行java文件时起到作用的是一下四个文件夹：bin、include、lib、jre。JDK包含JRE，而JRE包含JVM。bin：最主要的是javac（编译器）；include：java和JVM交互用的头文件；lib：类库；jre：java运行环境；

总的来说，JDK用于Java程序的开发，而JRE是运行class文件的运行环境不具备编译Java文件的功能。JDK提供Java开发人员使用，其中主要包含了Java的开发工具，其次也包括JRE。所以安装了JDK也就有了JRE。

* **三者的关系**

JVM：将字节码文件转换成具体系统平台的机器指令。

JRE：JVM+Java语言的核心类库。

JDK：JRE+Java开发工具包。

在实际开发的过程中：我们利用JDK（调用Java API）开发了属于我们自己的Java程序后，通过JDK中的编译程序（javac）将我们的java文件编译为java字节码文件（.class文件），然后在JRE上运行这些Java字节码文件，最后JVM解析这些字节码，映射到CPU指令集或OS的系统调用。

* **JVM的生命周期**

1、JVM实例对应了一个独立运行的java程序它是进程级别  
a) 启动。启动一个Java程序时，一个JVM实例就产生了，任何一个拥有public static void main(String[] args)函数的class都可以作为JVM实例运行的起点
b) 运行。main()作为该程序初始线程的起点，任何其他线程均由该线程启动。JVM内部有两种线程：守护线程和非守护线程，main()属于非守护线程，守护线程通常由JVM自己使用，java程序也可以表明自己创建的线程是守护线程  
c) 消亡。当程序中的所有非守护线程都终止时，JVM才退出；若安全管理器允许，程序也可以使用Runtime类或者System.exit()来退出  

2、JVM执行引擎实例则对应了属于用户运行程序的线程它是线程级别的

