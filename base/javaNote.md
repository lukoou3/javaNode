## 判断字符串是否为空
Blank用的较多，Blank会多判断空白字符，相当于比Empty多了trim。
```java
import org.apache.commons.lang3.StringUtils;

StringUtils.isBlank(str);
StringUtils.isEmpty(str);
```

## apache StringUtils常用方法
```java
import org.apache.commons.lang3.StringUtils;

boolean StringUtils.isBlank(str);			// " " 为 true
boolean StringUtils.isEmpty(str);			// " " 为 false
boolean StringUtils.isNotBlank(str);
boolean StringUtils.isNotEmpty(str);

String[] StringUtils.split(str,char/String,max);//切割字符串，使用给定字符 ;  max 主要是用来处理空内容的

String StringUtils.capitalize(str);			//首字母大写
String StringUtils.uncapitalize(str);		//首字母小写

boolean StringUtils.isNumeric(str);
boolean StringUtils.isNumericSpace(str);	// "" 为true "12 3" 为 true

String StringUtils.join(array/Iterable,chat/String);	// 类似于 js 的 join ,使用给定字符拼接数组或集合中的元素

String StringUtils.leftPad(str,size,padChar);		//给左边拼接固定长度的字符
String StringUtils.rightPad(str,size,padChar);		//右边拼接固定长度的字符
```

## 判断集合是否为空
判断集合为空（null或size=0）
```java
import org.apache.commons.collections.CollectionUtils;

CollectionUtils.isEmpty(coll);
```
## apache CollectionUtils常用方法
```java
import org.apache.commons.collections.CollectionUtils;

boolean CollectionUtils.isEmpty(Collection);
boolean CollectionUtils.isNotEmpty(Collection);
Collection CollectionUtils.union(a,b);			//并集
Collection CollectionUtils.intersection(a,b);	//交集
Collection CollectionUtils.disjunction(a,b);	//补集
Collection CollectionUtils.subtract(a,b);		//差集
```

## apache NumberUtils常用方法
```java
import org.apache.commons.lang3.math.NumberUtils;

int NumberUtils.toInt(String);
//还有 toLong,toDouble,toFloat,toShort 等

boolean NumberUtils.isDigits(String);		//判断是否全由数字组成 "" 为 false
boolean NumberUtils.isNumber(String);		// 支持 0x 类的表达形式
```

## 数组的复制
```java
//系统级别的native原生方法，效率高。参数含义是：（原数组， 原数组的开始位置， 目标数组， 目标数组的开始位置， 拷贝个数）
System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
//基于System.arraycopy。参数含义，（原数组，拷贝的个数）。
Arrays.copyOf(T[] original, int newLength)
//基于System.arraycopy。参数含义：（原数组，开始位置，结束位置）。
Arrays.copyOfRange(T[] original, int from, int to)
```

## 永远不要在循环之外调用wait方法
java中多线程中测试某个条件的变化用 if 还是用 while？

wait/notify、notifyAll都是和while配合应用的。可以避免多线程并发判断逻辑失效问题。

永远在while循环里而不是if语句下使用wait。这样，循环会在线程睡眠前后都检查wait的条件，并在条件实际上并未改变的情况下处理唤醒通知。

多线程消费或者多线程生产时候，当调到notify（）方法的时候，可能只叫醒同类的线性，说明白一点就是；生产者叫醒了另外一个生产者然后进入到wait中；解决办法把notify（）方法改成 notifyAll()。


