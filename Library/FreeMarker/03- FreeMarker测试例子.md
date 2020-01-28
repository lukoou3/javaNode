## FreeMarker测试例子

### 取值
```java
@Test
public void testShow() throws Exception {
    //创建配置类,这个configuration是freemarker提供的,不要导错包了
    Configuration configuration = new Configuration(Configuration.getVersion());
    String classpath = this.getClass().getResource("/").getPath();
    //设置模板路径
    configuration.setDirectoryForTemplateLoading(new File(classpath + "/templates/"));
    //设置字符集
    configuration.setDefaultEncoding("utf-8");

    //加载模板
    Template template = configuration.getTemplate("show.ftl");
    //数据模型
    Map map = new HashMap();
    People people = new People("小明", 246000, new Date(), 246000.12345D);
    map.put("people", people);

    //显示生成的数据
    Writer out = new OutputStreamWriter(System.out);
    template.process(map, out);
    out.flush();
}
```
show.ftl
```
name:${people.name}
age:${people.age}
birthday:${people.birthday?datetime}
money:${people.money}

Date的显示：
date：${people.birthday?date}
time：${people.birthday?time}
datetime：${people.birthday?datetime}

Date自定义格式显示，使用的是string()：
datetime：${people.birthday?string('yyyy-MM-dd')}
datetime：${people.birthday?string('hh:mm:ss')}
datetime：${people.birthday?string('yyyy-MM-dd hh:mm:ss')}

数字的显示：
<#--默认每三位使用逗号分隔-->
${people.age}
<#--如果不想显示为每三位分隔的数字，可以使用c函数将数字型转成字符串输出-->
${people.age?c}
```
输出：
```
name:小明
age:246,000
birthday:2020-1-28 21:48:38
money:246,000.123

Date的显示：
date：2020-1-28
time：21:48:38
datetime：2020-1-28 21:48:38

Date自定义格式显示，使用的是string()：
datetime：2020-01-28
datetime：09:48:38
datetime：2020-01-28 09:48:38

数字的显示：
246,000
246000
```

### ifelse
```java
@Test
public void testIfelse() throws Exception {
    //创建配置类,这个configuration是freemarker提供的,不要导错包了
    Configuration configuration = new Configuration(Configuration.getVersion());
    String classpath = this.getClass().getResource("/").getPath();
    //设置模板路径
    configuration.setDirectoryForTemplateLoading(new File(classpath + "/templates/"));
    //设置字符集
    configuration.setDefaultEncoding("utf-8");

    //加载模板
    Template template = configuration.getTemplate("ifelse.ftl");
    //数据模型
    Map map = new HashMap();
    map.put("name", "小明");

    //显示生成的数据
    Writer out = new OutputStreamWriter(System.out);
    template.process(map, out);
    out.flush();
}
```
ifelse.ftl
```
测试取出数据：
<#--${name}-->
hello ${name}.

测试if
<#--字符串相等可以直接使用==-->
<#if name == "小明">
hello ${name} 小明.
</#if>

测试if else
<#if name == "小花">
hello ${name}(if).
<#else>
hello ${name}(else).
</#if>

测试if elseif else
<#if name == "小花">
hello ${name}(if).
<#elseif name == "小明">
hello ${name}(elseif).
<#else >
hello ${name}(else).
</#if>

```
输出：
```
测试取出数据：
hello 小明.

测试if
hello 小明 小明.

测试if else
hello 小明(else).

测试if elseif else
hello 小明(elseif).
```

### List
```java
public void testList() throws Exception {
    //创建配置类,这个configuration是freemarker提供的,不要导错包了
    Configuration configuration = new Configuration(Configuration.getVersion());
    String classpath = this.getClass().getResource("/").getPath();
    //设置模板路径
    configuration.setDirectoryForTemplateLoading(new File(classpath + "/templates/"));
    //设置字符集
    configuration.setDefaultEncoding("utf-8");

    //加载模板
    Template template = configuration.getTemplate("list.ftl");
    //数据模型
    Map map = new HashMap();

    List<People> peoples = IntStream.range(1, 6)
            .mapToObj(i -> new People("小明" + i, 20 + i, new Date(), 1000 * i))
            .collect(Collectors.toList());

    map.put("peoples", peoples);

    //显示生成的数据
    Writer out = new OutputStreamWriter(System.out);
    template.process(map, out);
    out.flush();
}
```
list.ftl
```
people_index,people.name,people.age,people_has_next
<#--boolen不能自动的转化为string，用?c转一下-->
<#list peoples as people>
${people_index},${people.name},${people.age},${people_has_next?c}
</#list>

_index下标从0开始，可以在表达式中使用简单的计算
<#list peoples as people>
${people_index + 1},${people.name},${people.age},${people_has_next?c}
</#list>

_index和_has_next的使用
<#list peoples as people><#if people_index != 0>, </#if>'${people.name}'</#list>
<#list peoples as people>'${people.name}'<#if people_has_next>, </#if></#list>

通过下标访问list：
${peoples[0].name}
${peoples[1].name}

for循环的形式遍历list：
<#--
可以变通一下吗
<#list 1..100 as t>
    ${t}
</#list>
循环1到100
这不就相当于
for(int i=1;i<=100;i++){
}
-->
list长度：${peoples?size}
<#list 0..(peoples?size - 1) as i>
${peoples[i].name}
</#list>

使用break跳出迭代：
<#list peoples as people>
${people_index},${people.name},${people.age},${people_has_next?c}
<#if people_index == 1><#break></#if>
</#list>
```
输出：
```
people_index,people.name,people.age,people_has_next
0,小明1,21,true
1,小明2,22,true
2,小明3,23,true
3,小明4,24,true
4,小明5,25,false

_index下标从0开始，可以在表达式中使用简单的计算
1,小明1,21,true
2,小明2,22,true
3,小明3,23,true
4,小明4,24,true
5,小明5,25,false

_index和_has_next的使用
'小明1', '小明2', '小明3', '小明4', '小明5'
'小明1', '小明2', '小明3', '小明4', '小明5'

通过下标访问list：
小明1
小明2

for循环的形式遍历list：
list长度：5
小明1
小明2
小明3
小明4
小明5

使用break跳出迭代：
0,小明1,21,true
1,小明2,22,true
```

### Map
```java
public void testMap() throws Exception {
    //创建配置类,这个configuration是freemarker提供的,不要导错包了
    Configuration configuration = new Configuration(Configuration.getVersion());
    String classpath = this.getClass().getResource("/").getPath();
    //设置模板路径
    configuration.setDirectoryForTemplateLoading(new File(classpath + "/templates/"));
    //设置字符集
    configuration.setDefaultEncoding("utf-8");

    //加载模板
    Template template = configuration.getTemplate("map.ftl");
    //数据模型
    Map map = new HashMap();
    Map mapdata = new HashMap();
    mapdata.put("name", "小明");
    mapdata.put("people", new People("小明", 246000, new Date(), 246000.12345D));

    map.put("mapdata", mapdata);

    //显示生成的数据
    Writer out = new OutputStreamWriter(System.out);
    template.process(map, out);
    out.flush();
}
```
map.ftl
```
map通过['field']访问
${mapdata['name']}
${mapdata['people'].age}

map通过.field访问
${mapdata.name}
${mapdata.people.age}

map遍历：
<#list mapdata?keys as key>
${key_index}: ${key}: ${mapdata[key]}
</#list>
```
输出：
```
map通过['field']访问
小明
246,000

map通过.field访问
小明
246,000

map遍历：
0: name: 小明
1: people: People{name='小明', age=246000, birthday=Tue Jan 28 21:55:19 CST 2020, money=246000.12345}
```

### Null
```java
public void testNull() throws Exception {
    //创建配置类,这个configuration是freemarker提供的,不要导错包了
    Configuration configuration = new Configuration(Configuration.getVersion());
    String classpath = this.getClass().getResource("/").getPath();
    //设置模板路径
    configuration.setDirectoryForTemplateLoading(new File(classpath + "/templates/"));
    //设置字符集
    configuration.setDefaultEncoding("utf-8");

    //加载模板
    Template template = configuration.getTemplate("null.ftl");
    //数据模型
    Map map = new HashMap();
    People people = new People("小明", 246000, null, 246000.12345D);
    map.put("people1", people);

    //显示生成的数据
    Writer out = new OutputStreamWriter(System.out);
    template.process(map, out);
    out.flush();
}
```
null.ftl
```
<#if people??>
    ${people.name}
</#if>

<#if people1??>
    ${people1.name}
</#if>

<#--必须使用括号-->
<#if (people.name)??>
    ${people.name}
</#if>

<#if (people1.name)??>
    ${people1.name}
</#if>

<#--必须使用括号-->
<#if (people.name)!'' != ''>
    ${people.name}
</#if>


提供默认值：
<#--${(people.name)!'默认name'}表示，如果people或people.name为空显示默认。-->
${(people.name)!'默认name'}
${(people1.name)!'默认name'}
```
输出：
```

    小明


    小明



提供默认值：
默认name
小明
```

### StrFun
```java
public void testStrFun() throws Exception {
    //创建配置类,这个configuration是freemarker提供的,不要导错包了
    Configuration configuration = new Configuration(Configuration.getVersion());
    String classpath = this.getClass().getResource("/").getPath();
    //设置模板路径
    configuration.setDirectoryForTemplateLoading(new File(classpath + "/templates/"));
    //设置字符集
    configuration.setDefaultEncoding("utf-8");

    //加载模板
    Template template = configuration.getTemplate("strfun.ftl");
    //数据模型
    Map map = new HashMap();

    //显示生成的数据
    Writer out = new OutputStreamWriter(System.out);
    template.process(map, out);
    out.flush();
}
```
strfun.ftl
```
变量的定义：
<#assign stra = 'Hello' />
<#assign strb = 'world' />

${stra} ${strb}

字符串的操作：
<#--连接-->
${stra + strb}
<#--截取-->
${(stra + strb)?substring(5,8)}
<#--长度-->
${(stra + strb)?length}
<#--大写-->
${(stra + strb)?upper_case}
<#--小写-->
${(stra + strb)?lower_case}
<#--首字母大写-->
${strb?cap_first}
<#--index_of-->
${(stra + strb)?index_of('w')}
<#--last_index_of-->
${(stra + strb)?last_index_of('w')}
<#--contains-->
${(stra + strb)?contains('w')?c}
<#--replace-->
${(stra + strb)?replace('o','xx')}


截取字符串的简单写法[start..end](闭区间)：
${(stra + strb)?substring(5,8)}
${(stra + strb)[5]}
${(stra + strb)[5..7]}
```
输出：
```
变量的定义：

Hello world

字符串的操作：
Helloworld
wor
10
HELLOWORLD
helloworld
World
5
5
true
Hellxxwxxrld


截取字符串的简单写法[start..end](闭区间)：
wor
w
wor
```

### Opera
```java
public void testOpera() throws Exception {
    //创建配置类,这个configuration是freemarker提供的,不要导错包了
    Configuration configuration = new Configuration(Configuration.getVersion());
    String classpath = this.getClass().getResource("/").getPath();
    //设置模板路径
    configuration.setDirectoryForTemplateLoading(new File(classpath + "/templates/"));
    //设置字符集
    configuration.setDefaultEncoding("utf-8");

    //加载模板
    Template template = configuration.getTemplate("opera.ftl");
    //数据模型
    Map map = new HashMap();
    map.put("age", 23);

    //显示生成的数据
    Writer out = new OutputStreamWriter(System.out);
    template.process(map, out);
    out.flush();
}
```
opera.ftl
```
比较运算符使用gt、gte、lt、lte等：
<#--不是使用>，大部分时候，freemarker会把>解释成标签结束！ -->
<#if age gt 30>
gt 30
</#if>
<#if age gt 20>
gt 20
</#if>

逻辑运算符使用&&、||、!：
<#if age gte 10 && age lte 20>
age gte 10 && age lte 20
</#if>
<#if age gte 20 && age lte 30>
age gte 20 && age lte 30
</#if>
```
输出：
```
比较运算符使用gt、gte、lt、lte等：
gt 20

逻辑运算符使用&&、||、!：
age gte 20 && age lte 30
```

### udf
```java
public void testUdf() throws Exception {
    //创建配置类,这个configuration是freemarker提供的,不要导错包了
    Configuration configuration = new Configuration(Configuration.getVersion());
    String classpath = this.getClass().getResource("/").getPath();
    //设置模板路径
    configuration.setDirectoryForTemplateLoading(new File(classpath + "/templates/"));
    //设置字符集
    configuration.setDefaultEncoding("utf-8");

    //加载模板
    Template template = configuration.getTemplate("udf.ftl");
    //数据模型
    Map map = new HashMap();
    map.put("list", Arrays.asList(1,2,3,4,5));
    //感觉自定义函数不是很重要，这些操作完全可以在外面处理好再传到model中去
    map.put("mkStr", new MkStrMethodModelEx());

    //显示生成的数据
    Writer out = new OutputStreamWriter(System.out);
    template.process(map, out);
    out.flush();
}
```
udf.ftl
```
<#assign myList = [2,1,3,4,7,5]>
${mkStr(myList)}

${mkStr(list)}

${mkStr(list, ' #')}
```
输出：
```
2 ,1 ,3 ,4 ,7 ,5

1 ,2 ,3 ,4 ,5

1 #2 #3 #4 #5
```

### StringTemplateLoader
```java
//基于模板字符串加载模板
@Test
public void testStringTemplateLoader() throws Exception {
    //创建配置类
    Configuration configuration=new Configuration(Configuration.getVersion());
    //获取模板内容
    //模板内容，这里测试时使用简单的字符串作为模板
    String templateString= "name：${name}";

    //加载模板
    //模板加载器
    StringTemplateLoader stringTemplateLoader = new StringTemplateLoader();
    stringTemplateLoader.putTemplate("template",templateString);
    configuration.setTemplateLoader(stringTemplateLoader);
    Template template = configuration.getTemplate("template","utf-8");

    //数据模型
    Map map = new HashMap();
    map.put("name", "小明");

    //显示生成的数据
    Writer out = new OutputStreamWriter(System.out);
    template.process(map, out);
    out.flush();
}
```
输出：
```
名称：小明
```

