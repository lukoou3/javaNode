
## java - 将 Byte[] 读取为 unsigned short Java
我需要读取一个16位的字节数组作为unsigned short number，Java不支持unsigned short类型。
那我该怎么做呢？？请帮忙!!

最佳答案

假设您有来自非 Java 源的二进制数据，您必须读取和使用 Java 中的值: 将其读取为(带符号的)short，然后将其转换为 int，如下所示:

```java
int intVal = shortVal >= 0 ? shortVal : 0x10000 + shortVal 
```
您不能在 short 中表示 unsigned short 的所有值，但在 int 中可以。

关于java - 将 Byte[] 读取为 unsigned short Java，我们在Stack Overflow上找到一个类似的问题： https://stackoverflow.com/questions/7932701/

