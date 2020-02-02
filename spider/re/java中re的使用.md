## java中re的使用

### Pattern类和Matcher类
java.util.regex是一个用正则表达式所订制的模式来对字符串进行匹配工作的类库包。它包括两个类：Pattern和Matcher。 Pattern 一个Pattern是一个正则表达式经编译后的表现模式。 Matcher 一个Matcher对象是一个状态机器，它依据Pattern对象做为匹配模式对字符串展开匹配检查。

**Pattern类和Matcher类使用的基本流程**：  
* 1.导入java.util.regex包；    
* 2.根据你的string生成一个Pattern对象引用，例如string str = "abc+",  Pattern p = Pattern.compile(str)；    
* 3.调用Pattern的matcher方法，传入带匹配的字符串，生成Matcher对象引用；    
* 4.调用matcher类的诸多方法去实现查找和替换等功能。  

如下例所示：
```java
@Test
public void testPatternMatcher() {
    String strCharSequence = "abcabcabcdefabc";
    String strRegex = "(abc)+";
    Pattern p = Pattern.compile(strRegex);
    Matcher m = p.matcher(strCharSequence);
    while (m.find()) {
        System.out.println("Match \"" + m.group() + "\" at positions " + m.start() + "-" + (m.end() - 1));
    }
}
```
输出结果：
```
Match "abcabcabc" at positions 0-8
Match "abc" at positions 12-14
```

### Pattern类常用方法

#### static Pattern compile(String regex)
编译正则表达式生成Pattern对象。Pattern构造函数是私有的，只能这样创建。

此方法有一个重载的版本可以传入flags：
```java
//Compiles the given regular expression into a pattern with the given flags.
public static Pattern compile(String regex, int flags) {
    return new Pattern(regex, flags);
}
```
代码示例: 
```java
Pattern p = Pattern.compile("\\w+"); 
p.pattern();//返回 \w+ 
```
pattern() 返回正则表达式的字符串形式,其实就是返回Pattern.complile(String regex)的regex参数

##### flag的含义
这些标志都是的 Pattern类常量：

* CASE_INSENSITIVE: 匹配时大小写不敏感.    
* MULTILINE: 启用多行模式, `^ $`匹配行的开头和结尾.此外，`^`仍然匹配字符串的开始，`$`也匹配字符串的结束。默认情况下，这两个表达式仅仅匹配字符串的开始和结束。      
* DOTALL: 在此模式中元字符`.`可以匹配任意字符, 包括换行符.默认情况下，表达式'.'不匹配行的结束符。    
* COMMENTS: 在这种模式下，匹配时会忽略(正则表达式里的)空格字符(注：不是指表达式里的"//s"，而是指表达式里的空格，tab，回车之类)。注释从#开始，一直到这行结束。    
* UNICODE_CASE: 在这个模式下，如果你还启用了CASE_INSENSITIVE标志，那么它会对Unicode字符进行大小写不明感的匹配。默认情况下，大小写不明感的匹配只适用于US-ASCII字符集。  
* UNIX_LINES: 启用UNIX换行符, 在多行模式中使用`^ $`时只有`\n`被识别成终止符.  

在这些标志里面，Pattern.CASE_INSENSITIVE，Pattern.MULTILINE，以及Pattern.COMMENTS是最有用的(其中Pattern.COMMENTS还能帮我们把思路理清楚，并且/或者做文档)。注意，你可以用在表达式里插记号的方式来启用绝大多数的模式。这些记号就在上面那张表的各个标志的下面。你希望模式从哪里开始启动，就在哪里插记号。

可以用"OR" ('|')运算符把这些标志合使用

#### static boolean matches(String regex, CharSequence input)
是一个静态方法,用于快速匹配字符串,该方法适合用于只匹配一次,且匹配全部字符串.
```java
public static boolean matches(String regex, CharSequence input) {
    Pattern p = Pattern.compile(regex);
    Matcher m = p.matcher(input);
    return m.matches();
}
```
代码示例: 
```java
Pattern.matches("\\d+","2223");//返回true 
Pattern.matches("\\d+","2223aa");//返回false,需要匹配到所有字符串才能返回true,这里aa不能匹配到 
Pattern.matches("\\d+","22bb23");//返回false,需要匹配到所有字符串才能返回true,这里bb不能匹配到 
```

#### String[] split(CharSequence input)
Pattern有一个split(CharSequence input)方法,用于分隔字符串,并返回一个String[].

String.split(String regex)就是通过Pattern.split(CharSequence input)来实现的. 
```java
public String[] split(String regex) {
    return split(regex, 0);
}
public String[] split(String regex, int limit) {
    ...
    return Pattern.compile(regex).split(this, limit);
}
```

代码示例: 
```java
Pattern p=Pattern.compile("\\d+"); 
String[] str=p.split("我的QQ是:456456我的电话是:0532214我的邮箱是:aaa@aaa.com"); 
```
结果:str[0]="我的QQ是:" str[1]="我的电话是:" str[2]="我的邮箱是:aaa@aaa.com" 

#### Matcher matcher(CharSequence input)
终于轮到Matcher类登场了,Pattern.matcher(CharSequence input)返回一个Matcher对象.

Matcher类的构造方法也是私有的,不能随意创建,只能通过Pattern.matcher(CharSequence input)方法得到该类的实例.
 
Pattern类只能做一些简单的匹配操作,要想得到更强更便捷的正则匹配操作,那就需要将Pattern与Matcher一起合作.Matcher类提供了对正则表达式的分组支持,以及对正则表达式的多次匹配支持. 

代码示例: 
```java
Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("22bb23"); 
m.pattern();//返回p 也就是返回该Matcher对象是由哪个Pattern对象的创建的 
```

### Pattern类常用方法

#### 查找:matches()/ lookingAt()/ find()
Matcher类提供三个匹配操作方法,三个方法均返回boolean类型,当匹配到时返回true,没匹配到则返回false 

* matches()方法用来用来判断整个字符串是否匹配正则表达式，只有全部都匹配的时候才会返回true，否则返回false。但如果是部分匹配的话，则会移动匹配的开始的位置；    
* lookingAt()方法部分匹配，总是从第一个字符进行匹配,匹配成功了不再继续匹配，匹配失败了,也不继续匹配；    
* find()方法，部分匹配，从当前位置开始匹配，找到一个匹配的子串，将移动下次匹配的位置；    
* reset()方法，将当前开始匹配的位置重新设置到左边最开始的位置。  

可以结合下面的例子进行理解。
```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
 
public class IOTest {
    public static void main(String[] args){
        Pattern pattern = Pattern.compile("\\d{3,5}");
        String charSequence = "123-34345-234-00";
        Matcher matcher = pattern.matcher(charSequence);
 
        //虽然匹配失败，但由于charSequence里面的"123"和pattern是匹配的,所以下次的匹配从位置4开始
        print(matcher.matches());
        //测试匹配位置
        matcher.find();
        print(matcher.start());
 
        //使用reset方法重置匹配位置
        matcher.reset();
 
        //第一次find匹配以及匹配的目标和匹配的起始位置
        print(matcher.find());
        print(matcher.group()+" - "+matcher.start());
        //第二次find匹配以及匹配的目标和匹配的起始位置
        print(matcher.find());
        print(matcher.group()+" - "+matcher.start());
 
        //第一次lookingAt匹配以及匹配的目标和匹配的起始位置
        print(matcher.lookingAt());
        print(matcher.group()+" - "+matcher.start());
 
        //第二次lookingAt匹配以及匹配的目标和匹配的起始位置
        print(matcher.lookingAt());
        print(matcher.group()+" - "+matcher.start());
    }
    public static void print(Object o){
        System.out.println(o);
    }
}
```
输出结果：
```
false
4
true
123 - 0
true
34345 - 4
true
123 - 0
true
123 - 0
```

#### start()/ end()/ group()
当使用matches(),lookingAt(),find()执行匹配操作后,就可以利用以上三个方法得到更详细的信息. 

* start()返回匹配到的子字符串在字符串中的索引位置(包含).    
* end()返回匹配到的子字符串的最后一个字符在字符串中的索引位置(不包含).    
* group()返回匹配到的子字符串  

代码示例: 
```java
Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("aaa2223bb"); 
m.find();//匹配2223 
m.start();//返回3 
m.end();//返回7,返回的是2223后的索引号 
m.group();//返回2223 

Mathcer m2=m.matcher("2223bb"); 
m.lookingAt();   //匹配2223 
m.start();   //返回0,由于lookingAt()只能匹配前面的字符串,所以当使用lookingAt()匹配时,start()方法总是返回0 
m.end();   //返回4 
m.group();   //返回2223 

Matcher m3=m.matcher("2223bb"); 
m.matches();   //匹配整个字符串 
m.start();   //返回0,原因相信大家也清楚了 
m.end();   //返回6,原因相信大家也清楚了,因为matches()需要匹配所有字符串 
m.group();   //返回2223bb
```

#### start(int i)/end(int i)/group(int i)
start(),end(),group()均有一个重载方法它们是start(int i),end(int i),group(int i)专用于分组操作,Mathcer类还有一个groupCount()用于返回有多少组. 

子分组匹配的字符串在原字符串的位置为 [start(i),end(i)),左闭右开。

代码示例: 
```java
Pattern p=Pattern.compile("([a-z]+)(\\d+)"); 
Matcher m=p.matcher("aaa2223bb"); 
m.find();   //匹配aaa2223 
m.groupCount();   //返回2,因为有2组 
m.start(1);   //返回0 返回第一组匹配到的子字符串在字符串中的索引号 
m.start(2);   //返回3 
m.end(1);   //返回3 返回第一组匹配到的子字符串的最后一个字符在字符串中的索引位置. 
m.end(2);   //返回7 
m.group(1);   //返回aaa,返回第一组匹配到的子字符串 
m.group(2);   //返回2223,返回第二组匹配到的子字符串
```

#### 替换:replaceFirst()/replaceAll()
matcher里卖弄提供了replaceFirst、replaceAll、appendReplacement以及appendTail方法来进行替换有关的操作

* replaceAll方法 ：替换在原字符串中所有被正则表达式匹配的字串，并返回替换之后的结果    
* replaceFirst方法 ：替换在原字符串中第一个被正则表达式匹配的字串，并返回替换之后的结果    
* appendReplacement方法 ： 将当前匹配子串替换为指定字符串，并且将替换后的子串以及其之前到上次匹配子串之后的字符串段添加到一个StringBuffer对象里（需while(matcher.find())进行配合迭代）    
* appendTail(StringBuffer sb) 方法则将最后一次匹配工作后剩余的字符串添加到一个StringBuffer对象里。  

代码示例:
```java
//: strings/TheReplacements.java
import java.util.regex.*;

public class TheReplacements {
    private static final String CONSTSTRING = "fat cat, fat cat, fat cat, I like.";
  public static void main(String[] args) throws Exception {
      Pattern p = Pattern.compile("cat");
      Matcher m = p.matcher(CONSTSTRING);
      
      //replaceFirst()只替换匹配到的第一个地方
      System.out.println(m.replaceFirst("dog"));
      
      //replaceAll()替换所有匹配到的地方
      p = Pattern.compile("cat");
      m = p.matcher(CONSTSTRING);  
      System.out.println(m.replaceAll("dog"));
      
      //appendReplacement()将替换位置处及之前位置处的字符复制到StringBuffer中；
      //appendTail()将替换位置之后的字符复制到StringBuffer
      p = Pattern.compile("cat");
      m = p.matcher(CONSTSTRING);
      StringBuffer sb = new StringBuffer();
      while (m.find()) {
          m.appendReplacement(sb, "dog");
          System.out.println(sb.toString());
      }	 
      m.appendTail(sb);
      System.out.println(sb.toString());
  }
}
```
输出结果：
```
fat dog, fat cat, fat cat, I like.
fat dog, fat dog, fat dog, I like.
fat dog
fat dog, fat dog
fat dog, fat dog, fat dog
fat dog, fat dog, fat dog, I like.
```

### 分组引用和分组替换
分组引用用的都是`\1`和分组替换用的是`$1`，python中用的都是`\1`，不要搞混了。

示例：
```java
@Test
public void removeDuplicate() {
    String goal = "aabbcddeefgg";
    String regex = "(\\w)\\1+";
    Matcher matcher = Pattern.compile(regex).matcher(goal);
    String result = matcher.replaceAll("$1");
    Assert.assertEquals("abcdefg", result);
}
```

#### 关于正则表达式 \1 \2之类的问题
我们创建一个正则表达式
```java
var RegExp = /^(123)(456)\2\1$/;
```
这个正则表达式匹配到的字符串就是
123456456123
创建另外第二正则表达式
```java
var RegExp1 = /^(123)(456)\1$/;
```
这个正则表达式匹配到的字符串是
123456123
创建另外第三正则表达式
```java
var RegExp1 = /^(123)(456)\2$/;
```
这个正则表达式匹配到的字符串是
123456456

```
这个\1  \2......  都要和正则表达式集合()一起使用
简单的说就是
\1表示重复正则第一个圆括号内匹配到的内容
\2表示重复正则第二个圆括号内匹配到的内容
```