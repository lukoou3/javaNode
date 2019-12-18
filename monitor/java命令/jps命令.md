## jps命令使用
在Linux环境下显示一个进程的信息大家可能一直都在使用ps命令，比如用以下命令来显示当前系统执行的Java进程：
```sh
ps -ef | grep java
```
针对java的进程，jdk1.5以后提供了一个查看当前所有java进程pid的小工具。

位置：

`JAVA_HOME/bin/目录下面`

**jps(Java Virtual Machine Process Status Tool)**
是JDK 1.5提供的一个显示当前所有java进程pid的命令，简单实用，非常适合在linux/unix平台上简单察看当前java进程的一些简单情况。我们可以通过它来查看我们到底启动了几个java进程（因为每一个java程序都会独占一个java虚拟机实例），和他们的进程号，并可通过opt来查看这些进程的详细启动参数。  

格式：**`jps [-q] [-mlvV] [<hostid>]`**

具体 `[options]`选项解析：  
* -q：仅输出VM标识符(pid)，不包括classname,jar name,arguments in main method；   
* -m：输出main method的参数；   
* **-l**：输出完全的包名，应用主类名，jar的完全路径名；   
* -v：输出jvm参数 ；   
* -V：输出通过flag文件传递到JVM中的参数(.hotspotrc文件或-XX:Flags=所指定的文件 ；  


**其实平时最常用的就是jps、jps -l命令。**

**注：jps命令有个地方很不好，似乎只能显示当前用户的java进程，要显示其他用户的还是只能用unix/linux的ps命令。**