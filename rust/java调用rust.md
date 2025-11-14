
## 背景
之前调研questdb时发现它在java中调用rust来避免gc优化性能，所以就看了一下。

参考：
https://questdb.com/blog/leveraging-rust-in-our-high-performance-java-database/
https://github.com/questdb/rust-maven-plugin
https://www.jianshu.com/p/01596548867f

## java调用rust原理
就和java c代码一样，就是使用jni调用，questdb就只是写了一个rust-maven-plugin插件方便maven编译。


## 简单测试java调用rust
代码可以参考questdb和rust-maven-plugin的代码，rust-maven-plugin中有示例，更复杂的例子可以参考questdb的代码。

### 创建java maven工程

创建java-rust工程，在目录下创建rust目录，直接在rust目录下创建rust库工程，rust库工程该怎么写就怎么写，就是需要使用动态库的方式编译。
```
D:\IdeaWorkspace\java-rust\rust>cargo new jr --lib
```

java的maven配置， 里面的rust-maven-plugin就是编译rust，把文件复制到对应目录，起始完全可以自己编译把动态库放入java工程也行。jar-jni库是为了方便加载动态库。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.rust</groupId>
    <artifactId>java-rust</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.questdb</groupId>
            <artifactId>jar-jni</artifactId>
            <version>1.2.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.questdb</groupId>
                <artifactId>rust-maven-plugin</artifactId>
                <version>1.2.0</version>
                <executions>
                    <execution>
                        <id>rs-build</id>
                        <goals>
                            <goal>build</goal>
                        </goals>
                        <configuration>
                            <path>rust/jr</path>
                            <release>true</release>
                            <copyTo>${project.build.directory}/classes/com/rust/libs</copyTo>
                            <copyWithPlatformDir>true</copyWithPlatformDir>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.target}</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.0.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <!-- Do not copy the signatures in the META-INF folder.
                                    Otherwise, this might cause SecurityExceptions when using the JAR. -->
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

rust的配置：
```toml
[package]
name = "jr"
version = "0.1.0"
edition = "2024"

[lib]
crate-type = ["cdylib"]

[dependencies]
jni = "0.21.1"

```

### rust代码暴露jni接口
就创建了两个文件，似乎可以在任意个文件暴露接口，不是必须在lib文件暴露。

这里RustJniLib的hello和string方法就只是测试基本的方法调用，Decoder的方法测试了创建rust类，然后返回java rust中对象指针，通过指针调用rust方法，最后通过指针删除rust中的对象。

lib.rs：
```rust
mod decoder;

use jni::JNIEnv;
use jni::objects::{JClass, JString};
use jni::sys::jstring;


#[unsafe(no_mangle)]
pub extern "system" fn Java_com_rust_RustJniLib_hello(
    mut env: JNIEnv,
    _class: JClass,
    input: JString,
) -> jstring {
    // 从 Java 获取输入字符串
    let input: String = match env.get_string(&input) {
        Ok(s) => s.into(),
        Err(err) => {
            env.throw_new("java/lang/RuntimeException", format!("Failed to get string: {}", err))
                .expect("Failed to throw exception");
            return std::ptr::null_mut();
        }
    };

    // 创建输出字符串
    let output = format!("Hello from Rust: {}", input);
    match env.new_string(output) {
        Ok(s) => s.into_raw(),
        Err(err) => {
            env.throw_new("java/lang/RuntimeException", format!("Failed to create string: {}", err))
                .expect("Failed to throw exception");
            std::ptr::null_mut()
        }
    }
}

#[unsafe(no_mangle)]
pub extern "system" fn Java_com_rust_RustJniLib_reversedString(
    mut env: JNIEnv,
    _class: JClass,
    input: JString) -> jstring {
    let input: String =
        env.get_string(&input).expect("Couldn't get java string!").into();
    let reversed: String = input.chars().rev().collect();
    let reversed = format!("rust_reverse_pre: {}", reversed);
    let output = env.new_string(reversed)
        .expect("Couldn't create java string!");
    output.into_raw()
}

```


decoder.rs：
```rust
use jni::JNIEnv;
use jni::objects::{JClass, JString};
use jni::sys::jstring;

pub struct Decoder {
    pub id: i64
}

#[unsafe(no_mangle)]
pub extern "system" fn Java_com_rust_Decoder_create(
    mut env: JNIEnv,
    _class: JClass,
    id: i64,
) -> *mut Decoder {
    println!("create decoder {}", id);
    Box::into_raw(Box::new(Decoder{id}))
}

#[unsafe(no_mangle)]
pub extern "system" fn Java_com_rust_Decoder_destroy(
    mut env: JNIEnv,
    _class: JClass,
    decoder: *mut Decoder,
) {
    if decoder.is_null() {
        panic!("decoder pointer is null");
    }
    unsafe {
        let d = Box::from_raw(decoder);
        println!("destroy decoder {}", d.id);
        drop(d);
    }
}

#[unsafe(no_mangle)]
pub extern "system" fn Java_com_rust_Decoder_decode(
    mut env: JNIEnv,
    _class: JClass,
    decoder: *mut Decoder,
    input: JString,
) -> jstring {
    if decoder.is_null() {
        panic!("decoder pointer is null");
    }
    let input: String =
        env.get_string(&input).expect("Couldn't get java string!").into();
    let decoder = unsafe { &mut *decoder };
    let output = env.new_string(format!("{} decoded by rust decoder {}", input, decoder.id)).expect("Couldn't create java string!");
    output.into_raw()
}

```

### java jni接口

com.rust.RustJniLib:
```java
package com.rust;

import io.questdb.jar.jni.JarJniLoader;

public class RustJniLib {
    static {
        JarJniLoader.loadLib(
                RustJniLib.class,
                // A platform-specific path is automatically suffixed to path below.
                "/com/rust/libs",
                // The "lib" prefix and ".so|.dynlib|.dll" suffix are added automatically as needed.
                "jr");
    }

    public static native String hello(String input);
    public static native String reversedString(String str);
}
```

com.rust.Decoder:
```java
package com.rust;

public class Decoder {

    public static native long create(int id);

    public static native void destroy(long impl);

    public static native String decode(long impl, String input);
}

```

### 测试java调用jni
```java
public static void main(String[] args) {
    String result = hello("Java");
    System.out.println(result);
    result = reversedString("uf8字符串");
    System.out.println(result);
    long d1 = Decoder.create(1);
    long d2 = Decoder.create(2);
    System.out.println(d1);
    System.out.println(d2);
    String s1 = Decoder.decode(d1, "uf8字符串");
    String s2 = Decoder.decode(d2, "uf8字符串");
    System.out.println(s1);
    System.out.println(s2);
    Decoder.destroy(d1);
    Decoder.destroy(d2);
}
```

```
Hello from Rust: Java
rust_reverse_pre: 串符字8fu
create decoder 1
create decoder 2
1802401341104
1802401340784
uf8字符串 decoded by rust decoder 1
uf8字符串 decoded by rust decoder 2
destroy decoder 1
destroy decoder 2
```

### idea中运行代码问题
idea运行代码时使用的是自己的方式编译代码运行，会导致加载不到动态库，这里需要配置ant脚本，在编译前调用插件，rust-maven-plugin的github页面有说明。

创建rust-intellij.xml文件。在idea(View -> Tool Windows -> Ant)打开ant，配置rust-intellij.xml在编译前运行，之后运行代码就好了。

rust-intellij.xml：
```
<project name="rs-integration" default="rs-build" basedir=".">
    <description>
        IntelliJ integration to trigger maven steps to build the Rust code via the rust-maven-plugin.
    </description>
    <target name="rs-build">
        <exec executable="mvn.cmd">
            <arg value="org.questdb:rust-maven-plugin:build@rs-build"/>
        </exec>
    </target>
</project>
```

不知道为什么，这里必须配置mvn.cmd，mvn不行，可能是 Ant的问题。





