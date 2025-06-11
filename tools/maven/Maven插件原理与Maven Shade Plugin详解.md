
# Maven 插件原理与 Maven Shade Plugin 详解
此文为ai输出，提问：请为我介绍下maven插件的原理，然后为我详细介绍下Maven Shade Plugin插件

## 一、Maven 插件原理

Maven 是一个基于插件架构的构建工具，将构建过程分解为生命周期阶段（如compile、test、package），由插件执行具体任务。

1. 插件架构

* 定义：插件是独立模块，以 JAR 形式发布，包含实现特定功能的 Java 代码。    
* Mojo：插件的核心执行单元（Maven Plain Old Java Object），继承 org.apache.maven.plugin.AbstractMojo，实现 execute() 方法。    
* Mojo 配置：通过 @Mojo 注解或 META-INF/plexus/components.xml 定义目标（goal）、参数和绑定阶段。

2. 生命周期绑定

* Maven 生命周期包含阶段（如 clean、compile、package）。    
* 插件的 Mojo 通过 pom.xml 或默认配置绑定到阶段。    
* 示例：maven-compiler-plugin 的 compile Mojo 绑定到 compile 阶段。

3. 插件执行流程

* 解析 pom.xml，确定生命周期和绑定的插件目标。    
* 下载并加载插件 JAR 及其依赖。    
* 通过依赖注入将配置参数注入 Mojo。    
* 执行 Mojo 的 execute() 方法。    
* 输出日志或生成文件。

4. 配置机制

* 配置通过 pom.xml 的 configuration 节点或命令行参数提供。    
* 参数映射到 Mojo 类的字段（通过 @Parameter 注解）。    
* 示例：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.13.0</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```

5. 类加载与隔离

* Maven 使用独立类加载器加载插件，与核心运行时隔离。    
* 插件依赖由插件自身的 pom.xml 定义。

6. 扩展性

* 可开发自定义插件，继承 AbstractMojo，配置 META-INF/maven/plugin.xml 或使用注解。

7. 常见插件

* maven-compiler-plugin：编译代码。    
* maven-jar-plugin：打包 JAR。    
* maven-surefire-plugin：运行测试。    
* maven-shade-plugin：创建 fat JAR。


## 二、Maven Shade Plugin 详解

maven-shade-plugin用于创建包含所有依赖的可执行 JAR（fat JAR/uber JAR），支持类和资源合并、过滤、重定位，特别适合替换依赖中的类。

1. 插件概述

* GroupId：org.apache.maven.plugins    
* ArtifactId：maven-shade-plugin    
* 最新版本（截至 2023）：3.6.0    
* 功能：合并项目和依赖的类/资源到单个 JAR。重定位类以避免冲突。过滤类/资源，控制包含或排除。合并资源（如 META-INF 文件）。生成可执行 JAR。

2. 使用场景

* 创建可执行 JAR，简化部署。    
* 替换依赖中的类（你的需求）。    
* 解决依赖冲突（如同名类）。    
* 清理无用资源（如签名文件）。    
* 定制打包内容。

3. 配置结构

绑定到package阶段：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.6.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <!-- 配置项 -->
            </configuration>
        </execution>
    </executions>
</plugin>
```

4. 核心配置项

* outputFile：输出 JAR 路径，默认：target/${project.build.finalName}.jar。示例：xml`<outputFile>target/my-app.jar</outputFile>`    
* filters：控制依赖中类/资源的包含或排除。结构：

```xml
<filters>
    <filter>
        <artifact>groupId:artifactId</artifact>
        <includes>
            <include>path/to/include/**</include>
        </includes>
        <excludes>
            <exclude>path/to/exclude/**</exclude>
        </excludes>
    </filter>
</filters>
```
artifact：指定依赖（如 com.example:lib 或 *:*）。includes：保留文件（Ant 路径，如 com/example/**）。excludes：排除文件（如 com/example/Foo.class）。优先级：项目类 > includes 类 > 依赖类。    
* transformers：处理资源合并。常用 transformers：ManifestResourceTransformer：设置 Main-Class。ServicesResourceTransformer：合并 META-INF/services。AppendingTransformer：合并配置文件。示例：
```xml
<transformers>
    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
        <mainClass>com.example.Main</mainClass>
    </transformer>
</transformers>
```    
* relocations：重定位类/包到新命名空间。示例：
```xml
<relocations>
    <relocation>
        <pattern>com.example</pattern>
        <shadedPattern>com.myapp.example</shadedPattern>
    </relocation>
</relocations>
```    
* createDependencyReducedPom：生成精简 POM，移除合并的依赖（默认：true）。示例：`<createDependencyReducedPom>true</createDependencyReducedPom>`    
* minimizeJar：移除未使用类/资源，减小 JAR 大小。示例：`<minimizeJar>true</minimizeJar>`注意：可能移除必要类，需测试。    
* artifactSet：指定包含的依赖。示例：
```xml
<artifactSet>
    <includes>
        <include>com.example:lib</include>
    </includes>
</artifactSet>
```

5. 替换依赖中的类（你的需求）

步骤：

* 在项目中创建同名类（如 src/main/java/com/example/Foo.java），实现修改后的逻辑。    
* 配置 filters 排除依赖中的原类（可选，项目类默认优先）。    
* 示例配置：

```xml
<configuration>
    <filters>
        <filter>
            <artifact>com.example:lib</artifact>
            <excludes>
                <exclude>com/example/Foo.class</exclude>
            </excludes>
        </filter>
    </filters>
    <transformers>
        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <mainClass>com.example.Main</mainClass>
        </transformer>
    </transformers>
</configuration>
```    
* 运行 mvn clean package shade:shade，生成包含修改后类的 JAR。

6. 工作原理

* 合并：解压项目和依赖的 JAR，合并类/资源。    
* 过滤：应用 filters 规则，决定文件保留/排除。    
* 转换：通过 transformers 处理资源合并。    
* 重定位：修改类包名和字节码引用（relocations）。    
* 输出：生成 fat JAR 和精简 POM。

7. 注意事项

* 依赖冲突：项目类优先，filters 和 relocations 解决同名类冲突。    
* 签名问题：排除签名文件：
```xml
<filters>
    <filter>
        <artifact>*:*</artifact>
        <excludes>
            <exclude>META-INF/*.SF</exclude>
            <exclude>META-INF/*.DSA</exclude>
            <exclude>META-INF/*.RSA</exclude>
        </excludes>
    </filter>
</filters>
```    
* 性能：大型项目打包耗时，建议用 artifactSet 限制依赖。    
* 调试：用 mvn -X package shade:shade 查看日志。    
* 版本：推荐 3.6.0。

8. 高级用法

* 合并服务文件：
```xml
<transformers>
    <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
</transformers>
```    
* 自定义 Transformer：实现 ResourceTransformer 接口。    
* 多模块项目：在最终模块使用 shade 插件。

9. 总结

* Maven 插件：通过 Mojo 实现任务，绑定生命周期，依赖注入配置。    
* Shade Plugin：创建 fat JAR，支持类替换、资源合并、冲突解决。    
* 替换类：创建同名类，配置 filters 排除依赖类，运行 shade:shade。


笔记补充

* 运行命令：mvn clean package shade:shade    
* 调试日志：mvn -X package shade:shade    
* 替换类要点：确保 src/main/java 中类路径与依赖一致。验证 JAR 内容：jar tvf target/my-app.jar    
* 问题排查：类未替换：检查 filters 和类路径。签名错误：排除 META-INF 签名文件。依赖冲突：使用 relocations 或 excludes。

如果你有具体依赖（如groupId:artifactId）或类（如com.example.Foo），我可以进一步定制配置！

# maven-plugin-shade 详解
摘抄自：https://www.cnblogs.com/lkxed/p/maven-plugin-shade.html

maven-plugin-shade 插件提供了两个主要的能力：
1. 把整个项目（包含它的依赖）都打包到一个 "uber-jar" 中；
2. shade - 即重命名某些依赖的包。
具体来说，它提供了以下功能：
1. 按需选择要添加到最终 jar 包中依赖；
2. 重定位 class 文件；
3. 生成可执行 jar 包；
4. 生成项目资源文件。


## 一、介绍 [[1]](#fn1 "[1]")

> This plugin provides the capability to package the artifact in an uber-jar, including its dependencies and to shade - i.e. rename - the packages of some of the dependencies.

maven-plugin-shade 插件提供了**两个能力**：


* 把整个项目（包含它的依赖）都打包到一个 "uber-jar" 中    
* shade - 即重命名某些依赖的包


由此引出了**两个问题**：


* 什么是 uber-jar ？
uber-jar 也叫做 fat-jar 或者 jar-with-dependencies，意思就是包含依赖的 jar。    
* 什么是 shade ？
shade 意为遮挡，在此处可以理解为对依赖的 jar 包的重定向（主要通过重命名的方式）。


💁‍♂️下文中可能使用 shade 来代替 maven-plugin-shade。

## 二、基本使用 [[2]](#fn2 "[2]")


maven-plugin-shade 必须和 Maven 构建生命周期中的 package 阶段绑定，也就是说，当执行`mvn package`时会自动触发 shade。

要使用 maven-plugin-shade，只需要在 pom.xml 的`<plugins>`标签下添加它的配置即可，示例如下：


```xml
<project>
    ...
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.4</version>
                <configuration>
                    <!-- 此处按需编写更具体的配置 -->
                </configuration>
                <executions>
                    <execution>
                        <!-- 和 package 阶段绑定 -->
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    ...
</project>

```


默认情况下会把项目所有的依赖都包含进最终的 jar 包中。当然，我们也可以在`<configuration>`标签内配置更具体的规则。


## 三、常用功能


### 3.1 按需选择要添加到最终 jar 包中依赖 [[3]](#fn3 "[3]")


使用`<includes>`排除不需要的依赖，示例如下：


```xml
<configuration>
    <artifactSet>
        <excludes>
            <exclude>classworlds:classworlds</exclude>
            <exclude>junit:junit</exclude>
            <exclude>jmock:*</exclude>
            <exclude>*:xml-apis</exclude>
            <exclude>org.apache.maven:lib:tests</exclude>
            <exclude>log4j:log4j:jar:</exclude>
        </excludes>
    </artifactSet>
</configuration>

```


使用`<filters>`结合`<includes>`&`<excludes>`标签可实现更灵活的依赖选择，示例如下：


```xml
<configuration>
    <filters>
        <filter>
            <artifact>junit:junit</artifact>
            <includes>
                <include>junit/framework/**</include>
                <include>org/junit/**</include>
            </includes>
            <excludes>
                <exclude>org/junit/experimental/**</exclude>
                <exclude>org/junit/runners/**</exclude>
            </excludes>
        </filter>
        <filter>
            <artifact>*:*</artifact>
            <excludes>
                <exclude>META-INF/*.SF</exclude>
                <exclude>META-INF/*.DSA</exclude>
                <exclude>META-INF/*.RSA</exclude>
            </excludes>
        </filter>
    </filters>
</configuration>

```


除了可以通过自定义的 filters 来过滤依赖，此插件还支持自动移除项目中没有使用到的依赖，以此来最小化 jar 包的体积，只需要添加一项配置即可。示例如下：


```xml
<configuration>
    <minimizeJar>true</minimizeJar>
</configuration>

```

### 3.2 重定位 class 文件 [[4]](#fn4 "[4]")


如果最终的 jar 包被其他的项目所依赖的话，直接地引用此 jar 包中的类可能会导致类加载冲突，这是因为 classpath 中可能存在重复的 class 文件。为了解决这个问题，我们可以使用 shade 提供的重定位功能，把部分类移动到一个全新的包中。示例如下：


```xml
<configuration>
    <relocations>
        <relocation>
            <pattern>org.codehaus.plexus.util</pattern>
            <shadedPattern>org.shaded.plexus.util</shadedPattern>
            <excludes>
                <exclude>org.codehaus.plexus.util.xml.Xpp3Dom</exclude>
                <exclude>org.codehaus.plexus.util.xml.pull.*</exclude>
            </excludes>
        </relocation>
    </relocations>
</configuration>

```


涉及标签：


* `<pattern>`：原始包名    
* `<shadedPattern>`：重命名后的包名    
* `<excludes>`：原始包内不需要重定位的类，类名支持通配符


例如，在上述示例中，我们把 org.codehaus.plexus.util 包内的所有子包及 class 文件（除了 ~.xml.Xpp3Dom 和 ~.xml.pull 包下的所有 class 文件）重定位到了 org.shaded.plexus.util 包内。

当然，如果包内的大部分类我们都不需要，一个个排除就显得很繁琐了。此时我们也可以使用`<includes>`标签来指定我们仅需要的类，示例如下：


```xml
<project>
    ...
    <relocation>
        <pattern>org.codehaus.plexus.util</pattern>
        <shadedPattern>org.shaded.plexus.util</shadedPattern>
        <includes>
            <include>org.codehaud.plexus.util.io.*</include>
        </includes>
    </relocation>
    ...
</project>

```

### 3.3 生成可执行 jar 包 [[5]](#fn5 "[5]")


使用 maven-plugin-shade 后，最终生成的 jar 包可以包含所有项目所需要的依赖。我们会想，能不能直接运行这个 uber-jar 呢？答案是当然可以，并且十分简单，只需要指定`<mainClass>`启动类就可以了。示例如下：


```xml
<project>
    ...
    <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>org.sonatype.haven.HavenCli</mainClass>
            </transformer>
        </transformers>
    </configuration>
    ...
</project>

```


熟悉 jar 包的朋友们都知道，jar 包中默认会包含一个 MANIFEST.MF 文件，里面描述了一些 jar 包的信息。使用 java 自带的 jar 命令打包的时候可以指定 MANIFEST.MF，其中也可以指定 Main-Class 来使得 jar 包可运行。那么使用 shade 来指定和直接在 MANIFEST.MF 文件中指定有什么区别呢？


答案是没有区别，细心的读者会发现`<mainClass>`标签的父标签是`<transformer>`有一个 implementation 属性，其值为 "~.ManifestResourceTransformer"，意思是 Manifest 资源文件转换器。上述示例只自指定了启动类，因此 shade 会为我们自动生成一个包含 Main-Class 的 MANIFEST.MF 文件，然后在打 jar 包时指定这个文件。


那如果我们想要完全定制 MANIFEST.MF 文件内容怎么办呢？我们可以使用`<manifestEntries>`标签，示例如下：


```xml
<project>
    ...
    <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <manifestEntries>
                    <Main-Class>org.sonatype.haven.ExodusCli</Main-Class>
                    <Build-Number>123</Build-Number>
                </manifestEntries>
            </transformer>
        </transformers>
    </configuration>
    ...
</project>

```


### 4.4 生成资源文件 [[6]](#fn6 "[6]")

项目中涉及到的依赖可能会有它们所必需的资源文件，使用 shade 可以把它们聚合在同一个 jar 包中。


默认地，shade 为我们提供了 12 个 ResourceTransformer 类：

| 类名                                                                                                                                                                                                 | 作用                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| [ApacheLicenseResourceTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#ApacheLicenseResourceTransformer "ApacheLicenseResourceTransformer")       | 防止 LICENSE 文件重复                                   |
| [ApacheNoticeResourceTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#ApacheNoticeResourceTransformer "ApacheNoticeResourceTransformer")          | 准备合并的 NOTICE                                       |
| [AppendingTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#AppendingTransformer "AppendingTransformer")                                           | 为某个资源文件附加内容                                  |
| [ComponentsXmlResourceTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#ComponentsXmlResourceTransformer "ComponentsXmlResourceTransformer")       | 聚合 Plexus `components.xml`                            |
| [DontIncludeResourceTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#DontIncludeResourceTransformer "DontIncludeResourceTransformer")             | 防止包含指定的资源                                      |
| [GroovyResourceTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#GroovyResourceTransformer "GroovyResourceTransformer")                            | 合并 Apache Groovy 的扩展模块                           |
| [IncludeResourceTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#IncludeResourceTransformer "IncludeResourceTransformer")                         | 添加项目中的文件为资源文件                              |
| [ManifestResourceTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#ManifestResourceTransformer "ManifestResourceTransformer")                      | 自定义 MANIFEST 文件                                    |
| [PluginXmlResourceTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#PluginXmlResourceTransformer "PluginXmlResourceTransformer")                   | 聚合 Maven 的 plugin.xml 配置                           |
| [ResourceBundleAppendingTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#ResourceBundleAppendingTransformer "ResourceBundleAppendingTransformer") | 合并 ResourceBundles                                    |
| [ServicesResourceTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#ServicesResourceTransformer "ServicesResourceTransformer")                      | 重定位且合并 META-INF/services 资源文件中的 class 文件. |
| [XmlAppendingTransformer](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html#XmlAppendingTransformer "XmlAppendingTransformer")                                  | 为 XML 资源文件附加内容                                 |


每种 ResourceTransformer 的具体使用可以点击对应链接查看官方示例，这里不再赘述。


如果上述 12 个类都不能够满足我们的需求，我们可以实现 shade 提供的接口，按需自定义一个 ResourceTransformer，实现方法详见官网[Using your own Shader implementation](http://maven.apache.org/plugins/maven-shade-plugin/examples/use-shader-other-impl.html "Using your own Shader implementation")。


## 参考来源


* [http://maven.apache.org/plugins/maven-shade-plugin/index.html](http://maven.apache.org/plugins/maven-shade-plugin/index.html "http://maven.apache.org/plugins/maven-shade-plugin/index.html") [↩︎](#fnref1 "↩︎")    
* [http://maven.apache.org/plugins/maven-shade-plugin/usage.html](http://maven.apache.org/plugins/maven-shade-plugin/usage.html "http://maven.apache.org/plugins/maven-shade-plugin/usage.html") [↩︎](#fnref2 "↩︎")    
* [http://maven.apache.org/plugins/maven-shade-plugin/examples/includes-excludes.htm](http://maven.apache.org/plugins/maven-shade-plugin/examples/includes-excludes.htm "http://maven.apache.org/plugins/maven-shade-plugin/examples/includes-excludes.htm") [↩︎](#fnref3 "↩︎")    
* [http://maven.apache.org/plugins/maven-shade-plugin/examples/class-relocation.html](http://maven.apache.org/plugins/maven-shade-plugin/examples/class-relocation.html "http://maven.apache.org/plugins/maven-shade-plugin/examples/class-relocation.html") [↩︎](#fnref4 "↩︎")    
* [http://maven.apache.org/plugins/maven-shade-plugin/examples/executable-jar.html](http://maven.apache.org/plugins/maven-shade-plugin/examples/executable-jar.html "http://maven.apache.org/plugins/maven-shade-plugin/examples/executable-jar.html") [↩︎](#fnref5 "↩︎")    
* [http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html](http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html "http://maven.apache.org/plugins/maven-shade-plugin/examples/resource-transformers.html") [↩︎](#fnref6 "↩︎")


# Maven Shade Plugin 使用详细说明
摘抄自：https://blog.csdn.net/Li_WenZhang/article/details/142874764

`maven-shade-plugin`是一个Maven 插件，用于将项目及其所有依赖项打包成一个可执行的胖 JAR（fat JAR）。这个插件不仅可以合并多个依赖，还支持过滤、排除、修改 MANIFEST 文件等高级功能。本文将深入介绍`maven-shade-plugin`的使用方法和配置技巧。

## 一、什么是 maven-shade-plugin

`maven-shade-plugin`是ApacheMaven 提供的一个插件，用于将项目的所有依赖项打包成一个可执行的胖 JAR 文件。这种 JAR 包包含所有项目所需的依赖项，可以在不额外配置的情况下直接运行。在构建微服务、发布可执行应用时，使用胖 JAR 可以减少部署复杂度。


## 二、maven-shade-plugin 的基本用法


* **引入插件**
 在项目的 `pom.xml` 文件中引入 `maven-shade-plugin`，并在 `package` 阶段执行： 
 
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.4.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```    
* **指定 Main Class** 在 `configuration` 节点中设置主类，使生成的 JAR 文件可以直接通过 `java -jar` 命令运行： 

```xml
<configuration>
    <transformers>
        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <mainClass>com.example.MainClass</mainClass>
        </transformer>
    </transformers>
</configuration>
```


## 三、排除签名文件防止“Invalid signature file digest for Manifest main attributes”错误


在运行胖 JAR 文件时，可能会遇到`Invalid signature file digest for Manifest main attributes`错误。这通常是因为 JAR 包中的`META-INF/*.SF`、`META-INF/*.DSA`、`META-INF/*.RSA`等签名文件导致的。为了解决这个问题，可以使用`maven-shade-plugin`排除这些文件：


```xml
<configuration>
    <filters>
        <filter>
            <artifact>*:*</artifact>
            <excludes>
                <exclude>META-INF/*.SF</exclude>
                <exclude>META-INF/*.DSA</exclude>
                <exclude>META-INF/*.RSA</exclude>
            </excludes>
        </filter>
    </filters>
</configuration>
```


这段配置可以过滤所有签名文件，避免出现“Invalid signature file digest for Manifest main attributes”错误。


## 四、处理依赖冲突与资源合并

在合并依赖项时，可能会出现资源冲突，比如多个依赖包含同样的资源文件。这时可以使用`transformers`来解决冲突。


* **合并特定资源文件** 使用 `AppendingTransformer` 可以合并特定的资源文件，如 `META-INF/spring.factories`： 

```xml
<transformers>
    <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
        <resource>META-INF/spring.factories</resource>
    </transformer>
    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
        <mainClass>com.example.MainClass</mainClass>
    </transformer>
</transformers>
```    
* **重命名包路径避免类冲突** 使用 `relocations` 可以将某些包路径重命名，以防止不同依赖中的类冲突： 

```xml
<relocations>
    <relocation>
        <pattern>com.example.some.library</pattern>
        <shadedPattern>com.example.shaded.some.library</shadedPattern>
    </relocation>
</relocations>
```

## 五、完整配置示例

以下是一个整合了过滤、合并和重命名的`maven-shade-plugin`配置示例：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.4.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <filters>
                            <filter>
                                <artifact>*:*</artifact>
                                <excludes>
                                    <exclude>META-INF/*.SF</exclude>
                                    <exclude>META-INF/*.DSA</exclude>
                                    <exclude>META-INF/*.RSA</exclude>
                                </excludes>
                            </filter>
                        </filters>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring.factories</resource>
                            </transformer>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>com.example.MainClass</mainClass>
                            </transformer>
                        </transformers>
                        <relocations>
                            <relocation>
                                <pattern>com.example.some.library</pattern>
                                <shadedPattern>com.example.shaded.some.library</shadedPattern>
                            </relocation>
                        </relocations>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

此配置确保生成的胖 JAR 文件没有签名文件，能够直接运行且不受类冲突影响。


## 六、注意事项

在使用`maven-shade-plugin`时，有几个关键点需要注意，以确保打包过程顺利并获得预期效果。以下将详细阐述使用胖 JAR 文件可能遇到的问题及其解决方法：

### 1. 胖 JAR 文件体积较大

打包成胖 JAR 文件的主要特点是它将所有依赖项整合到一个单一的 JAR 文件中。这在部署和分发时确实便捷，但也带来了一些潜在问题：


* **存储空间**：胖 JAR 包含所有依赖项，体积会大幅增加。对于资源受限的环境（例如嵌入式设备或低带宽场景），胖 JAR 可能会占用过多的磁盘空间和传输资源。这种情况下，可以考虑将一些公共依赖项单独发布，以减少胖 JAR 的体积。    
* **内存使用**：加载胖 JAR 时，JVM 会将整个文件载入内存。若项目依赖众多且规模庞大，会导致启动时占用较多内存。对此，可以通过优化依赖和精简项目来缓解。    
* **不适合大型项目**：当项目中有大量依赖项或 JAR 文件时，胖 JAR 的体积可能会非常庞大，这会影响构建和传输速度。因此，在大型项目中，可以使用轻量级打包方式（例如 Spring Boot 提供的依赖分层打包）来替代胖 JAR。


### 2. 构建时间增加

`maven-shade-plugin`在打包过程中会将项目及其所有依赖项合并成一个胖 JAR 文件，这一过程通常会消耗较多时间：


* **打包性能问题**：打包过程中的每一步（如签名文件过滤、资源合并和包路径重命名）都可能导致构建时间延长。特别是在项目依赖项较多时，构建时间可能会显著增加。可以使用 `mvn package -DskipTests` 命令跳过测试步骤以加快打包速度。    
* **构建过程调优**：为减少打包时间，可尝试减少无关紧要的依赖项，或使用缓存功能（如 Maven 的本地仓库缓存），尽量减少对网络依赖的重新下载。    
* **增量构建**：如果项目频繁构建，考虑使用增量构建工具或分层构建策略，确保每次只构建修改过的部分，从而减少整体构建时间。


### 3. 资源冲突和合并问题

在胖 JAR 文件中，不同的依赖可能包含相同的资源文件（如配置文件、属性文件等），这会导致资源冲突。使用`maven-shade-plugin`时，必须配置合理的`transformers`来处理资源冲突问题：

* **常见冲突资源**：某些配置文件，如 `META-INF/spring.factories`、`META-INF/services/*` 或日志配置文件（如 `logback.xml`），经常会在依赖项中重复出现，导致冲突。为避免问题，可以使用 `AppendingTransformer` 合并此类文件。    
* **冲突排查与定位**：当胖 JAR 出现无法正常运行的情况（如类加载异常或配置未生效），检查 JAR 内部资源冲突通常是解决问题的关键。可以使用工具（如 jar-utility）解压胖 JAR 并手动检查冲突资源。    
* **适当过滤与合并**：使用过滤器（`filters`）排除不必要的资源文件，可以有效减少资源冲突。对于必须合并的资源文件，确保配置正确的 `transformers` 类型。需要注意的是，一些依赖可能包含必需的配置文件，排除或合并不当会导致程序运行异常。

### 4. 运行时依赖的外部资源

胖 JAR 文件虽然封装了大部分依赖项，但仍有一些资源或配置文件可能需要外部访问：


* **外部配置文件**：尽管胖 JAR 自带配置文件，但在某些场景下，使用外部配置文件（例如 Spring Boot 的外部化配置）可以提高灵活性。通过在 `java -jar` 命令中指定 `--spring.config.location` 参数，可以加载外部配置。    
* **依赖于外部资源的库**：一些依赖库需要额外的文件或服务支持（如数据库驱动程序或证书文件），这些资源可能无法封装到胖 JAR 中。确保此类资源在部署环境中可用，以避免运行时异常。

### 5. 使用 FAT JAR 的替代方案

尽管胖 JAR 带来了便利，但在某些场景下，可能有更合适的替代方案来满足项目需求：

* **模块化 JAR**：对于 JDK 9 及更高版本的项目，可以考虑使用模块化 JAR 来管理项目依赖。模块化 JAR 能更精细地控制依赖关系，减少资源浪费。    
* **Spring Boot Thin Launcher**：适合 Spring Boot 项目，该工具可以在保持胖 JAR 部署便利性的同时，减少 JAR 文件体积。Thin Launcher 会将依赖项单独存储并按需加载，避免了胖 JAR 的冗余。    
* **Docker 容器化**：对于分布式系统或微服务，Docker 容器是一种更合适的解决方案。将应用程序打包到 Docker 容器中可以保证依赖项的隔离性，减少资源冲突，适用于大规模部署。

## 七、总结

`maven-shade-plugin`是一个功能强大的插件，可以帮助你将项目及其依赖打包成一个胖 JAR 文件。在实际使用中，通过合并资源、排除签名文件和重命名包路径，可以构建出无冲突的可执行 JAR，从而简化部署流程。




