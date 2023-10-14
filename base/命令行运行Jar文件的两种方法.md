## 命令行运行Jar文件的两种方法
jar包：test.jar 

主类：org.duomu.demo.HelloWorld

### 含有Main-Class的jar
使用java -jar 命令
```
java –jar test.jar
```

### 没有Main-Class的jar
另一个比较简单的解决方法是，直接用命令行将jar文件包含在你的classpath环境变量下，具体地键入以下命令并执行：java –classpath test.jar org.duomu.demo.HelloWorld，不用配置清单文件即可成功运行。
```
java –classpath test.jar org.duomu.demo.HelloWorld
```

后来我在linux中试了怎么不行，-cp行：
```
java -cp flink-libs-with-dependencies.jar -Xmx4096m -Xms2048m com.flink.test.JsonLogParseStream --local 1 --parallelism 1 --json_format json
```







 
 
