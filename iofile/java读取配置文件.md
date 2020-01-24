## java读取配置文件
java读取配置文件建议使用类加载器从classspath下读取文件，不要使用绝对路径
![getResourceAsStream](/assets/getResourceAsStream.png)

### Java中getResourceAsStream的用法
Java中的getResourceAsStream有以下几种： 

* 1. Class.getResourceAsStream(String path) ： path 不以’/'开头时默认是从此类所在的包下取资源，以’/'开头则是从ClassPath根下获取。其只是通过path构造一个绝对路径，最终还是由ClassLoader获取资源。 

* 2. Class.getClassLoader.getResourceAsStream(String path) ：默认则是从ClassPath根下获取，path不能以’/'开头，最终是由ClassLoader获取资源。 

* 3. ServletContext. getResourceAsStream(String path)：默认从WebAPP根目录下取资源，Tomcat下path是否以’/'开头无所谓，当然这和具体的容器实现有关。 

* 4. Jsp下的application内置对象就是上面的ServletContext的一种实现。 

### Properties对象读取资源文件
```java
Properties properties= new Properties(); 
InputStream in = DemoTest2.class.getClassLoader().getResourceAsStream("demo0.properties");
// 加载输入流对象
properties.load(in);
String value = properties.getProperty("name");
```
下面的文件目录图，是项目中文件的位置信息；下面的例子是按照这个图来演示的。
```
.
|-- java
|   |-- ibard
|   |   |-- demo1
|   |   |   `-- DemoTest1.java
|   |   `-- demo2
|   |       `-- DemoTest2.java
`-- resources
    |-- demo0.properties
    `-- ibard
        |-- demo2
        |   `-- demo2.properties
        `-- demo3
            `-- demo3.properties
```
项目打包发布后的目录结构：(要注意的是，我们操作的文件状态是下面这个目录的情形！)
```
target
|-- classes
|   |-- demo0.properties
|   |-- ibard
|   |   |-- demo1
|   |   |   `-- DemoTest1.java
|   |   `-- demo2
|   |       `-- DemoTest2.java
|   |   |   `-- demo2.properties
|   |   |-- demo3
|   |       `-- demo3.properties
```
#### 生成properties 对象
首先，我们需要生成Properties 对象：Properties properties= new Properties(); 。

#### 获取资源文件的输入流
然后，我们需要得到资源文件的输入流，而得到输入流的方法有两种。

##### Concrete.class.getResourceAsStream(String path)
path可以是相对路径，也可以是绝对路径。
```java
InputStream in = DemoTest2.class.getResourceAsStream("demo2.properties");
```

##### Concrete.class.getClassLoader().getResourceAsStream(String path)
path只能是绝对路径。但是绝对路径却不以/ 开头。
```java
InputStream in = DemoTest2.class.getClassLoader().getResourceAsStream("ibard/demo3/demo3.properties");
```

#### 加载输入流，获取值
在得到输入流之后，将输入流加载到Properties 对象，之后就可以获取值了。
```java
// 加载输入流对象
properties.load(in);
String value = properties.getProperty("name");
```



