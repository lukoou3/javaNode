## FreeMarker基本使用
freemarker模板中使用freemarker的指令，关于freemarker的指令需要知道：
```
1、注释，即<#‐‐和‐‐>，介于其之间的内容会被freemarker忽略
2、插值（Interpolation）：即${..}部分,freemarker会用真实的值代替${..}
3、FTL指令：和HTML标记类似，名字前加#予以区分，Freemarker会解析标签中的表达式或逻辑。
4、文本，仅文本信息，这些不是freemarker的注释、插值、FTL指令的内容会被freemarker忽略解析，直接输出内
容。
```

### 数据类型
#### 一、	直接指定值
直接指定值可以是字符串、数值、布尔值、集合及Map对象。
1. 字符串
直接指定字符串值使用单引号或双引号限定。字符串中可以使用转义字符”\"。如果字符串内有大量的特殊字符，则可以在引号的前面加上一个字母r，则字符串内的所有字符都将直接输出。

2. 数值
数值可以直接输入，不需要引号。FreeMarker不支持科学计数法。

3. 布尔值 
直接使用true或false，不使用引号。

4. 集合
集合用中括号包括，集合元素之间用逗号分隔。
使用数字范围也可以表示一个数字集合，如1..5等同于集合[1, 2, 3, 4, 5]；同样也可以用5..1来表示[5, 4, 3, 2, 1]。

5. Map对象
Map对象使用花括号包括，Map中的key-value对之间用冒号分隔，多组key-value对之间用逗号分隔。
注意：Map对象的key和value都是表达式，但key必须是字符串。

6. 时间对象
root.put("date1", new Date());
${date1?string("yyyy-MM-dd HH:mm:ss")}

7. JAVABEAN的处理
Freemarker中对于javabean的处理跟EL表达式一致，类型可自动转化！非常方便！

#### 二、	输出变量值
FreeMarker的表达式输出变量时，这些变量可以是顶层变量，也可以是Map对象的变量，还可以是集合中的变量，并可以使用点（.）语法来访问Java对象的属性。

1. 顶层变量
所谓顶层变量就是直接放在数据模型中的值。输出时直接用${variableName}即可。

2. 输出集合元素
可 以根据集合元素的索引来输出集合元素，索引用中括号包括。如： 输出[“1”， “2”， “3”]这个名为number的集合，可以用${number[0]}来输出第一个数字。FreeMarker还支持用number[1..2]来表示原 集合的子集合[“2”， “3”]。

3. 输出Map元素
对于JavaBean实例，FreeMarker一样把它看作属性为key，属性值为value的Map对象。
输出Map对象时，可以使用点语法或中括号语法，如下面的几种写法的效果是一样的：
```
book.author.name                        
book.author["name"]                             
book["author"].name 
book["author"]["name"]
```
使用点语法时，变量名字有和顶层变量一样的限制，但中括号语法没有任何限制。

### 数据类型常见示例
直接指定值 
   字符串 ： "Foo"或 者'Foo'或"It's \"quoted\""或r"C:\raw\string" 
   数字：123.45 
   布尔值：true, false 
   序列：["foo", "bar", 123.45], 1..100 
   哈希表：{"name":"green mouse", "price":150} 
   检索变量    顶层变量：user 
   从哈希表中检索数据：user.name, user[“name”] 
   从序列中检索：products[5] 
   特殊变量：.main 
   字符串操作 
   插值（或连接）："Hello ${user}!"（或"Free" + "Marker"） 
   获取一个字符：name[0] 
   序列操作 
   连接：users + ["guest"] 
   序列切分：products[10..19]  或  products[5..] 
   哈希表操作 
   连接：passwords + {"joe":"secret42"} 
   算数运算: (x * 1.5 + 10) / 2 - y % 100 
   比 较 运 算 ： x == y,   x != y,   x < y,   x > y,   x >= y,   x <= y等等 
    逻辑操作：!registered && (firstVisit || fromEurope) 
    内建函数：name?upper_case 
    方法调用：repeat("What", 3) 
   处理不存在的值 
   默认值：name!"unknown"  或者(user.name)!"unknown"  或者
name!  或者  (user.name)! 
   检测不存在的值：name?? 或者(user.name)?? 

### 内建函数
FreeMarker提供了一些内建函数来转换输出，可以在任何变量后紧跟?，?后紧跟内建函数，就可以通过内建函数来转换输出变量。

常用的内建函数
```
length： 字符串的长度
size： 获得集合中元素的个数；
c： 转成字符串，例如输出int和boolean时使用
cap_first 首字母变大写
lower_case： 将字符串转成小写；
upper_case： 将字符串转成大写；
ends_with 以...结尾
contains 包含
date 日期格式化
time 日期格式化
datetime 日期格式化
？string('0.##') 把字符串变成两位小数
is_string is_number 判断变量是不是string是不是numb
```

### 核心指令

#### 取值指令
Freemarker中通过${}方式取值
```
常用${var}语法进行取值
对null、不存在对象取值${var!}
取包装对象的值，通过“点”语法：${User.name}
取值的时候进行计算、赋值
Date类型格式${date?string('yyyy-MM-dd')}
如何转义HTML内容：${var?html}
```

boolean类型值的format
```
布尔值：${booleanVar?string('yes','no')}
```

date类型值的format
```
日期：${dateVar?string('yyyy-MM-dd')}
```

null或者不存在的变量取值
```
null：${nullVar!'我是默认值'}
missing：${ssssVar!'我是默认值'}
```

#### 变量的赋值和运算
Freemarker中支持变量运算
```
<#assign var = 100>
var = <font color="#ff5142">${var}</font><br>
var + 100 = <font color="#ff5142">${var+100}</font>
```

#### if指令
```
<#assign var = 999>
<#if var == 999>
    var = 999
<#elseif var == 888>
    var = 888
<#else >
    var = 111
</#if>
```

#### List和Map的遍历
```
<#list list as item>
    ${item}<br/>
</#list>
<#list map?keys as key>
    ${key}:${map[key]}<br/>
</#list>
```

### 算术运算符
FreeMarker表达式中支持“+”、“－”、“*”、“/”、“%”运算符。

### 比较运算符
表达式中支持的比较运算符有如下几种：
```
1. =（或者==）： 判断两个值是否相等；
2. !=： 判断两个值是否不相等；注： =和!=可以用作字符串、数值和日期的比较，但两边的数据类型必须相同。而且FreeMarker的比较是精确比较，不会忽略大小写及空格。
3. >（或者gt）： 大于
4. >=（或者gte）： 大于等于
5. <（或者lt）： 小于
6. <=（或者lte）： 小于等于
```

注： 上面这些比较运算符可以用于数字和日期，但不能用于字符串。大部分时候，使用gt比>有更好的效果，因为FreeMarker会把>解释成标签的结束字符。可以使用括号来避免这种情况，如：`<#if (x>y)>`。

### 逻辑运算符
```
1. &&： 逻辑与；
2. ||： 逻辑或；
3. !： 逻辑非
```
逻辑运算符只能用于布尔值。

### 空值处理
FreeMarker的变量必须赋值，否则就会抛出异常。而对于FreeMarker来说，null值和不存在的变量是完全一样的，因为FreeMarker无法理解null值。

FreeMarker提供两个运算符来避免空值：

1. !： 指定缺失变量的默认值；  
2. ??：判断变量是否存在。  

!运算符有两种用法：variable!或variable!defaultValue。第一种用法不给变量指定默认值，表明默认值是空字符串、长度为0的集合、或长度为0的Map对象。
使用!运算符指定默认值并不要求默认值的类型和变量类型相同。
```
测试空值处理：
<#-- ${sss} 没有定义这个变量，会报异常！ -->
${sss!} <#--没有定义这个变量，默认值是空字符串！ -->
${sss!"abc"} <#--没有定义这个变量，默认值是字符串abc！ -->
```
如果是嵌套对象则需要使用（）括起来。
例： `${(stu.bestFriend.name)!''}`表示，如果stu或bestFriend或name为空默认显示空字符串。


??运算符返回布尔值，如：variable??，如果变量存在，返回true，否则返回false。
```
<#if stus??>
<#list stus as stu>
......
</#list>
</#if>
```

### 自定义函数
步骤一：编写自定义函数类
```java
import freemarker.template.TemplateMethodModelEx;
import freemarker.template.TemplateModelException;
import freemarker.template.TemplateSequenceModel;

import java.util.List;

public class MkStrMethodModelEx implements TemplateMethodModelEx{
    @Override
    public Object exec(List arguments) throws TemplateModelException {
        /*SimpleSequence simpleSequence = (SimpleSequence) arguments.get(0);
        //SimpleSequence 是Freemarker里的数据类型代表数组或list，Java从Freemarker取值时不能直接强转
        //必须先用Freemarker的数据类型取出来然后进行转换，toList是过时的方法却没有找到替代的方法
        List lists = simpleSequence.toList();
        return lists.stream().map(x -> x.toString()).collect(Collectors.joining(" ,"));*/

        /**
         * 上面的SimpleSequence只匹配<#assign myList = [2,1,3,4,7,5,8,6,9]>
         * java.util.List匹配的是DefaultListAdapter（通过debugger查看运行时类型得知）
         * TemplateSequenceModel是上面两个的公共接口
         */
        TemplateSequenceModel sequenceModel = (TemplateSequenceModel) arguments.get(0);
        String sep = arguments.size() > 1? arguments.get(1).toString() :" ,";
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < sequenceModel.size(); i++) {
            if(i != 0)
                sb.append(sep);
            sb.append(sequenceModel.get(i).toString());
        }
        return sb.toString();
    }
}
```
步骤二：添加自定义函数类，并指定方法名
```java
//数据模型
Map map = new HashMap();
map.put("list", Arrays.asList(1,2,3,4,5));
//感觉自定义函数不是很重要，这些操作完全可以在外面处理好再传到model中去
map.put("mkStr", new MkStrMethodModelEx());
```
步骤三：使用自定义函数
```
<#assign myList = [2,1,3,4,7,5]>
${mkStr(myList)}

${mkStr(list)}

${mkStr(list, ' #')}
```




