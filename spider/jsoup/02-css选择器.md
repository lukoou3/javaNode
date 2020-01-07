## css选择器
用类似于CSS或jQuery的语法来查找元素。

使用Element.select(String selector) 和 Elements.select(String selector) 方法实现，选择器过滤的思想和jQuery一样。

Element、Elements类的select函数返回的都是Elements，匹配不到返回的空的Elements而不是null，所以不比担心NPE，Jsoup的选择器完全可以类比jquery

Elements first()返回列表的第一个元素或者null，要注意匹配不到的时候不检查会产生NullPointerException

Elements toString() = outerHtml() 匹配不到元素输出为空字符串

## 选择器语法(Selector syntax)
css选择器的语法可以去org.jsoup.select.Selector类的注释上看，很详细

A selector is a chain of simple selectors, separated by combinators. Selectors are case insensitive (including against elements, attributes, and attribute values).

The universal selector (*) is implicit when no element selector is supplied (i.e. *.header and .header is equivalent).

选择器可以使用链式语法，选择器匹配不区分大小写（包括元素、属性、属性值）。   
通用选择器是隐式的，即`*.header`和`.header`是等效的

### 基本选择器
基本选择器和jQuery的差不多

* `#id`：id选择器  
* `element`：标签选择器  
* `ns|element`：通过标签在命名空间查找元素，比如：可以用 `fb|name` 语法来查找 `<fb:name>` 元素  
* `*|element`：匹配所有命名空间下的元素  
* `.class`：类选择器  
* `*`：通用选择器  
* `selector1,selector2,selectorN`：组合选择器  

### 属性选择器
属性选择器和jQuery的差不多，不过这里没有[attr!=val]选择器

这里面有一个支持正则的强大选择器

这里面例如[attr=val]的形式val可以不加引号，也可以加单引号和双引号

匹配不区分大小写

* `[attr]`：包含属性attr  
* `[^attrPrefix]`：包含attrPrefix开头的属性  
* `[attr=val]`：属性相等  
* `[attr="val"]`：属性相等(加单引号和双引号都行)  
* `[attr^=valPrefix]`：开头匹配  
* `[attr$=valSuffix]`：结尾匹配  
* `[attr*=valContaining]`：属性包含  
* `[attr~=regex]`：匹配正则，正则的匹配类似Python中re的search，并不是从头开始匹配的  
* `[selector1][selector2][selectorN]`：复合属性选择器，需要同时满足多个条件时使用。  

### 层级选择器
有四个层级选择器，这四个层级选择器和jQuery中的一模一样：

* `ancestor descendant`：后代选择器    
* `parent > child`：子选择器    
* `prev + next`：下一个同级元素    
* `prev ~ siblings`：后面所有的同级元素

**注意：这里没有选择前一个和前面的同级元素的选择器，可以利用Elements的方法实现**

### 基本的伪选择器(Pseudo selectors)
* `:eq(n)`：elements whose sibling index is equal to n    
* `:lt(n)`：elements whose sibling index is less than n    
* `:gt(n)`：elements whose sibling index is greater than n

注意上面英文解释的**sibling index**，jsoup的eq、lt、gt和jQuery时有区别的，soup的eq索引根据的是子元素在父元素中的索引，而jQuery中指的是匹配到的元素列表的索引。
Elements的eq(index)函数则是直接调用了集合的get方法，这点和jQuery一样。


* `:has(selector)`：elements that contains at least one element matching the selector	div:has(p) finds divs that contain p elements    
* `:not(selector)`：elements that do not match the selector. See also Elements.not(String)	div:not(.logo) finds all divs that do not have the "logo" class.   

has和not的用法和jQuery一样：has保留包含特定后代的元素，去掉那些不含有指定后代的元素。not删除与指定表达式匹配的元素

### 匹配文本的伪选择器(Pseudo selectors)
带不带Own的区别：是否只判断自己的文本节点，contains和matches的区别：matches使用正则匹配。

* `:contains(text)`：指的是元素是否包含特定的文本，不区分大小写，文本包括所有后代元素的文本，相当于Text()方法的返回值是否包含特定的文本。elements that contains the specified text. The search is case insensitive. The text may appear in the found element, or any of its descendants.    
* `:matches(regex)`：和 `:contains(text)` 一样, 只是支持正则表达式。elements whose text matches the specified regular expression. The text may appear in the found element, or any of its descendants.    
* `:containsOwn(text)`：只判断元素自己的文本是否包含，后代元素的文本的不算。elements that directly contain the specified text. The search is case insensitive. The text must appear in the found element, not any of its descendants.    
* `:matchesOwn(regex)`：只判断own text。elements whose own text matches the specified regular expression. The text must appear in the found element, not any of its descendants.    
* `:empty`：匹配所有不包含子元素或者文本的空元素

### 结构化的伪选择器(Structural pseudo selectors)
这些选择器都是见名知意的。

* `:nth-child(an+b)`：    
* `:nth-of-type(an+b)`：    
* `:first-child`：    
* `:last-child`：    
* `:first-of-type`：    
* `:last-of-type`：    
* `:only-child`：    
* `:only-of-type`：    
* `:nth-last-child(an+b)`：    
* `:nth-last-of-type(an+b)`：

nth-child(n)表示第几个子元素，nth-child(an+b)表示n取值0、1、3...。

tr:nth-child(2n+1) finds every odd row of a table. :nth-child(10n-1) the 9th, 19th, 29th, etc, element. li:nth-child(5) the 5h li

## css选择器测试代码
```java
package com;


import org.apache.commons.io.FileUtils;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;
import org.junit.Before;
import org.junit.Test;

import java.io.File;
import java.util.function.Function;
import java.util.stream.Collectors;

public class selectElementTest {
	private String menuHtml;
	private String contentHtml;

	@Before
	public void init() throws Exception {
		menuHtml = FileUtils.readFileToString(new File("bookmenu.html"));
		contentHtml = FileUtils.readFileToString(new File("bookcontent.html"));
	}

	@Test
	public void test(){
		Document doc = Jsoup.parse(menuHtml);
		Elements elements = doc.select("div#noexist").select("div");
		/**
		 * class Elements extends ArrayList<Element>
		 * Element、Elements类的select函数返回的都是Elements，匹配不到返回的空的Elements而不是null，所以不比担心NPE，Jsoup的选择器完全可以类比jquery
		 * Elements.first()返回列表的第一个元素或者null，要注意匹配不到的时候不检查会产生NullPointerException
		 * toString() = outerHtml() 匹配不到元素输出为空字符串
		 */
		System.out.println("1" + elements + "1");
		System.out.println(doc.select("div#noexist div").first());
	}

	@Test
	public void test1(){
		Document doc = Jsoup.parse(menuHtml);
		/**
		 * css选择器的语法可以去org.jsoup.select.Selector类的注释上看，很详细
		 *
		 * :last-child和:last-of-type的区别
		 * last-child代表子元素的最后一个元素
		 * last-of-type代表子元素中某个类型元素中的最后一个元素
		 *
		 * 同理的还有：
		 * first-child、first-of-type
		 * only-child、only-of-type
		 * nth-child(an+b)、nth-of-type(an+b)
		 * 这些结构化的选择器索引都是子元素或者同辈元素的索引
		 */
		Elements elements = doc.select("div.listmain dt:last-of-type ~ dd");
		System.out.println(elements.size());
		doc.select("div.listmain dt:last-child");//选不到，父元素的最后一个元素是dd
	}

	@Test
	public void test2(){
		Document doc = Jsoup.parse(menuHtml);
		/**
		 * 有四个层级选择器：
		 * E F	an F element descended from an E element
		 * E > F	an F direct child of E
		 * E + F	an F element immediately preceded by sibling E
		 * E ~ F	an F element preceded by sibling E
		 *
		 * 这四个层级选择器和jQuery中的一模一样：
		 * ancestor descendant
		 * parent > child
		 * prev + next
		 * prev ~ siblings
		 *
		 * 注意：这里没有选择前一个和前面的同级元素的选择器
		 */
		Elements elements = doc.select("div.listmain dt:last-of-type + dd");
		System.out.println(elements.size());
		System.out.println(elements.text());
	}

	@Test
	public void testAttr() {
		Document doc = Jsoup.parse(menuHtml);
		/**
		 * [attr]
		 * [^attrPrefix]
		 * [attr=val]
		 * [attr="val"]
		 * [attr^=valPrefix]
		 * [attr$=valSuffix]
		 * [attr*=valContaining]
		 * [attr~=regex]
		 *
		 * 属性选择器和jQuery的差不多，不过这里没有[attr!=val]选择器
		 * 这里面有一个支持正则的强大选择器
		 * 这里面例如[attr=val]的形式val可以不加引号，也可以加单引号和双引号
		 */
		Elements elements = doc.select("div[class]");
		printElementsClass(elements);

		elements = doc.select("div[class=logo]");
		printElementsClass(elements);
		System.out.println(elements.first() == doc.select("div[class='logo']").first());

		elements = doc.select("div[class^=l]");
		printElementsClass(elements);
		elements = doc.select("div[class^='l']");
		printElementsClass(elements);

		elements = doc.select("div[class$=k]");
		printElementsClass(elements);

		elements = doc.select("div[class*=l]");
		printElementsClass(elements);

		//正则的匹配类似Python中re的search，并不是从头开始匹配的
		elements = doc.select("div[class~=[a-z]{1,3}]");
		printElementsClass(elements);
		elements = doc.select("div[class~=^[a-z]{1,3}]");//和上面的效果一样
		printElementsClass(elements);
		elements = doc.select("div[class~=^[a-z]{1,3}$]");
		printElementsClass(elements);

		//Jsoup没有jQuery的[attribute!=value]，但是提供了:not(selector)伪选择器和not函数
		elements = doc.select("div:not(div[class*=l])");
		printElementsClass(elements);
		elements = doc.select("div").not("div[class*=l]");
		printElementsClass(elements);
	}

	@Test
	public void testIndex() {
		Document doc = Jsoup.parse(menuHtml);
		/**
		 * 经过测试，Jsoup的eq、lt、gt和jQuery时有区别的
		 * Jsoup的eq索引根据的是子元素在父元素中的索引，而jQuery中指的是匹配到的元素列表的索引
		 * eq(index)函数则是直接调用了集合的get方法，这点和jQuery一样
		 * 这样一来Jsoup的eq似乎就和nth-child(an+b)对应了，这里面没有child(n)选择器，nth-child的索引是从1开始的
		 * nth-child(n)表示第几个子元素，nth-child(an+b)表示n取值0、1、3...。
		 * tr:nth-child(2n+1) finds every odd row of a table. :nth-child(10n-1) the 9th, 19th, 29th, etc, element. li:nth-child(5) the 5h li
		 */
		Elements elements = doc.select("div.listmain dt:last-of-type ~ dd:eq(16)");
		printElementsText(elements);
		printElementsText(doc.select("div.listmain dt:last-of-type ~ dd").select("dd:eq(16)"));
		printElementsText(doc.select("div.listmain dt:last-of-type ~ dd").eq(2));
		elements = doc.select("div.listmain dt:last-of-type ~ dd:lt(2)");
		printElementsText(elements);
		elements = doc.select("div.listmain dt:last-of-type ~ dd:gt(732)");
		printElementsText(elements);

		printElementsText(doc.select("div.listmain a:eq(0)"));

		doc = Jsoup.parse("<table>\n" +
				"  <th><td>Header 1</td></th>\n" +
				"  <tr><td>Value 1</td></tr>\n" +
				"  <tr><td>Value 2</td></tr>\n" +
				"</table><table>\n" +
				"  <th><td>Header 1</td></th>\n" +
				"  <tr><td>Value 1</td></tr>\n" +
				"  <tr><td>Value 2</td></tr>\n" +
				"</table>");
		elements = doc.select("tr:eq(1)");
		printElementsText(elements);
		printElementsText(doc.select("table tr:eq(1)"));

		System.out.println("##############################");

		printElementsText(doc.select("table:eq(0) tr:nth-child(2n+1)"));
		printElementsText(doc.select("table:eq(0) tr:nth-child(2n)"));
		printElementsText(doc.select("table:eq(0) tr:nth-child(1)"));
		printElementsText(doc.select("table:eq(0) tr:nth-child(2)"));
	}

	@Test
	public void testHasNot() {
		Document doc = Jsoup.parse(menuHtml);
		/**
		 * has和not的用法和jQuery一样：
		 * has保留包含特定后代的元素，去掉那些不含有指定后代的元素。
		 * not删除与指定表达式匹配的元素
		 * :has(selector)	elements that contains at least one element matching the selector	div:has(p) finds divs that contain p elements
		 * :not(selector)	elements that do not match the selector. See also Elements.not(String)	div:not(.logo) finds all divs that do not have the "logo" class.
		 */
		Elements elements = doc.select("div.listmain dd:has(:contains(第七百三十四章))");
		printElementsText(elements);
		elements = doc.select("div.listmain dd:has(:contains('第七百三十四章'))");
		printElementsText(elements);

		//这里匹配到一个元素是因为dd前面还有一个同辈的dt，gt判断的是子元素的索引
		elements = doc.select("div.listmain dd:not(:gt(1))");
		printElementsText(elements);
	}

	@Test
	public void testContains() {
		Document doc = Jsoup.parse(menuHtml);
		/**
		 * :contains(text)  指的是元素是否包含特定的文本，不区分大小写，文本包括所有后代元素的文本，相当于Text()方法的返回值是否包含特定的文本
		 * elements that contains the specified text. The search is case insensitive. The text may appear in the found element, or any of its descendants.
		 *
		 * :matches(regex)	和 :contains(text) 一样, 只是支持正则表达式
		 * elements whose text matches the specified regular expression. The text may appear in the found element, or any of its descendants.
		 *
		 * :containsOwn(text) 只判断元素自己的文本是否包含，后代元素的文本的不算
		 * elements that directly contain the specified text. The search is case insensitive. The text must appear in the found element, not any of its descendants.
		 *
		 * :matchesOwn(regex) 只判断own text
		 * elements whose own text matches the specified regular expression. The text must appear in the found element, not any of its descendants.
		 *
		 */
		Elements elements = doc.select("div:contains(第七百三十四章)");
		System.out.println(elements.size());
		elements = doc.select("div:contains(class)");//匹配不到，判断的是文本内容
		printElementsText(elements);

		elements = doc.select("div.listmain dd:matches(第.{1,6}章)");
		System.out.println(elements.size());
	}

	@Test
	public void testStructuralPseudo () {
		Document doc = Jsoup.parse(menuHtml);

		/**
		 * :nth-child(an+b)
		 * :nth-of-type(an+b)
		 * :first-child
		 * :last-child
		 * :first-of-type
		 * :last-of-type
		 * :only-child
		 * :only-of-type
		 *
		 * :nth-last-child(an+b)
		 * :nth-last-of-type(an+b)
		 *
		 * :empty 匹配所有不包含子元素或者文本的空元素
		 */
		Elements elements = doc.select("*:empty");
		System.out.println(elements.size());
	}

	public void printElementsText(Elements elements){
		printElements(elements, e -> e.text());
	}

	public void printElementsAttr(Elements elements, final String attr){
		printElements(elements, e -> e.attr(attr));
	}

	public void printElementsClass(Elements elements){
		printElements(elements, e -> e.attr("class"));
	}

	public void printElements(Elements elements, Function<Element, String> func){
		System.out.println(elements.stream().map(func).collect(Collectors.joining(" ,")));
	}
}
```







