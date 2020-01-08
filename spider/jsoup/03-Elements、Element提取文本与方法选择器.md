## 03-Elements、Element提取文本与方法选择器
Elements其实就是Element的集合对象，Elements继承自`ArrayList<Element>`。对Elements的method都会作用到每个Element上，选择器的方法返回的都是Elements对象。Elements类的存在为CSS以及jQuery选择器的语法提供了实现。

Element提取文本的几个方法：
```java
String attr(String attributeKey)  提取元素属性
String text() 会返回元素及其后代的文本，会处理空白字符，元素之间的文本会以空格分隔
String ownText() 只提取自己的文本标签，遇到br会补一个空格
String wholeText() 对所有的文本标签里的文本原样连接输出。if you do not want the text to be normalized.
List<TextNode> textNodes() 只返回自己的文本节点，不包括后代元素的。
String outerHtml() 提取HTML，带元素本身的标签
String html() 元素里面的html，和jQuery的一样
```
以上一个方法如果不满足我们提取文本的需求，我们完成可以可以遍历Element自定义提取文本的规则，仿照着jsoup的源码写就行。jsoup对HTML提取封装的非常好，所有信心都在Element对象中，想提取哪个文本节点就能提取哪个文本节点。

text()格式化空格的方法：

* 文本标签的多个空白会被格式化成一个   
* 如果是block Tag 文本之间如果没有空格会补上空格，br也会补上空格  
* 如果是inline Tag不会在各个文本之间补上空格  


```java
    String html = "<p>One <span>Two</span> Three <br> Four</p>";
    Document doc = Jsoup.parse(html);
    Element element = doc.select("p").first();
    System.out.println(element.text());//"One Two Three Four"
    System.out.println(element.ownText());//One Three Four"
    System.out.println(element.children());//Elements[<span>, <br>]
    System.out.println(element.childNodes());//List<Node>["One ", <span>, " Three ", <br>, " Four"]
    System.out.println(element.textNodes());//List<TextNode>["One ", " Three ", " Four"]
```

Element类的方法选择器：
```java
/***
 * nextElementSibling()返回后一个同级元素
 * previousElementSibling()返回前一个同级元素
 * siblingElements()返回所有的同级元素（父元素的所有子元素），除了元素自己
 * firstElementSibling() 父元素的所有子元素的第一个
 * lastElementSibling() 父元素的所有子元素的最后一个
 * nextSibling、previousSibling这些方法名中不带Element的返回的是Node
 */
Document doc = Jsoup.parse(html);
Element element = doc.select("p").first();
element.nextElementSibling();
element.nextSibling();
element.previousElementSibling();
element.previousSibling();
element.siblingElements();
element.firstElementSibling();
element.lastElementSibling();
```

Elements类提取文本：
```java
/**
 * String text()连接每个Element的element.text()，以空格分隔
 * List<String> eachText() 每个Element的element.text()放入集合，过滤空元素
 *
 * 看到这里我觉得Elements除了select方法，别的方法不看也行，Elements就是Element的list，详细看看Element的方法即可
 * Elements的操作我们可以自己实现，而且用java8甚至scala的语法更加灵活。
 */
Document doc = Jsoup.parse(html);
String text = doc.select("p").text();
System.out.println(text);
List<String> texts = doc.select("p").eachText();
System.out.println(texts);
```

Elements类的方法选择器，这里的选择器方法竟然还比Element和css选择器的多，可以选择前面的同辈元素：
```java
/**
 * Elements的prev、next、prevAll、nextAll都是分别对每个元素查找，
 * 字符串选择器只有next、nextAll的语法，函数的方式更多，而且速度更快，这点和jQuery一样
 *  prev、next、prevAll、nextAll 都有一个可选的参数用于过滤
 */
Document doc = Jsoup.parse(html);
//Get the immediate previous element sibling of each element in this list.
Elements elements = doc.select("p").prev();
System.out.println(elements.size());

//Get the immediate previous element sibling of each element in this list, filtered by the query.
elements = doc.select("p").prev("[class=title]");
System.out.println(elements.size());

//Get all of the previous element siblings of each element in this list.
elements = doc.select("p").prevAll();
System.out.println(elements.size());

//Get the immediate next element sibling of each element in this list.
elements = doc.select("p").next();
System.out.println(elements.size());
```
和提取文本的时候一样，当默认的方法无法满足时，我们可以自己实现。

### 测试代码
提取文本
```java
package com;

import org.apache.commons.io.FileUtils;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.nodes.TextNode;
import org.junit.Before;
import org.junit.Test;

import java.io.File;
import java.util.List;

public class selectTextTest {
	private String menuHtml;
	private String contentHtml;

	@Before
	public void init() throws Exception {
		menuHtml = FileUtils.readFileToString(new File("bookmenu.html"));
		contentHtml = FileUtils.readFileToString(new File("bookcontent.html"));
	}

	@Test
	public void test(){
		/**
		 * 官方的一个小示例
		 * 提取元素属性 String attr(String attributeKey)
		 * 提取元素文本 String text() 会返回元素及其后代的文本，会处理空白字符，元素之间的文本会已空格分隔
		 * Gets the combined text of this element and all its children. Whitespace is normalized and trimmed.
		 * 提取HTML String outerHtml() 带元素本身的标签
		 * 			String html() 元素里面的html，和jQuery的一样
		 */
		String html = "<p>An <a href='http://example.com/'><b>example</b></a> link.</p>";
		Document doc = Jsoup.parse(html);
		Element link = doc.select("a").first();

		String text = doc.body().text(); // "An example link"
		String linkHref = link.attr("href"); // "http://example.com/"
		String linkText = link.text(); // "example""

		String linkOuterH = link.outerHtml();
		// "<a href="http://example.com"><b>example</b></a>"
		String linkInnerH = link.html(); // "<b>example</b>"
	}

	@Test
	public void testElementText(){
		/**
		 * 文本标签的多个空白会被格式化成一个
		 * 如果是block Tag 文本之间如果没有空格会补上空格，br也会补上空格
		 * 如果是inline Tag不会在各个文本之间补上空格
		 * @see #wholeText() if you don't want the text to be normalized.
		 * @see #ownText()
		 * @see #textNodes()
		 */
		String html = "<p>An<a href='http://example.com/'><b>example</b></a>link.</p>";
		Document doc = Jsoup.parse(html);
		String text = doc.select("p").first().text();
		System.out.println(text);//Anexamplelink.

		html = "<p> An   <a href='http://example.com/'><b>  example  </b></a>   link.</p>";
		doc = Jsoup.parse(html);
		text = doc.select("p").first().text();
		System.out.println(text);//An example link.

		html = "<p>An  <a href='http://example.com/'><b>  example  </b></a>   link.</p>";
		doc = Jsoup.parse(html);
		text = doc.select("p").first().text();
		System.out.println(text);//An example link.

		html = "<div>An<div>example</div>link.</div>";
		doc = Jsoup.parse(html);
		text = doc.select("div").first().text();
		System.out.println(text);//An example link.

		html = "<div> An <div>   example   </div>   link.  </div>";
		doc = Jsoup.parse(html);
		text = doc.select("div").first().text();
		System.out.println(text);//An example link.

		html = "<div>An<br/>example<br/>link.</div>";
		doc = Jsoup.parse(html);
		text = doc.select("div").first().text();
		System.out.println(text);//An example link.

		html = "<div>An<p>example</p><p>link.</p></div>";
		doc = Jsoup.parse(html);
		text = doc.select("div").first().text();
		System.out.println(text);//An example link.
	}

	@Test
	public void testElementOwnText() {
		/**
		 * ownText()
		 * 只提取自己的文本标签，遇到br会补一个空格
		 */
		String html = "<p>An<a href='http://example.com/'><b>example</b></a>link.</p>";
		Document doc = Jsoup.parse(html);
		String text = doc.select("p").first().ownText();
		System.out.println(text);//Anlink.

		html = "<p>An<br/><a href='http://example.com/'><b>example</b></a>link.</p>";
		doc = Jsoup.parse(html);
		text = doc.select("p").first().ownText();
		System.out.println(text);//An link.

		html = "<div>An<p>example</p>link.</div>";
		doc = Jsoup.parse(html);
		text = doc.select("div").first().ownText();
		System.out.println(text);//Anlink.
	}

	@Test
	public void testElementWholeText() {
		/**
		 * wholeText()
		 * 对所有的文本标签里的文本原样连接输出
		 */
		String html = "<p>An<a href='http://example.com/'><b>example</b></a>link.</p>";
		Document doc = Jsoup.parse(html);
		String text = doc.select("p").first().wholeText();
		System.out.println(text);//Anexamplelink.

		html = "<p> An   <a href='http://example.com/'><b>  example  </b></a>   link.</p>";
		doc = Jsoup.parse(html);
		text = doc.select("p").first().wholeText();
		System.out.println(text);//An example link.


		html = "<div>An<div>example</div>link.</div>";
		doc = Jsoup.parse(html);
		text = doc.select("div").first().wholeText();
		System.out.println(text);//An example link.

		html = "<div> An <div>   example   </div>   link.  </div>";
		doc = Jsoup.parse(html);
		text = doc.select("div").first().wholeText();
		System.out.println(text);//An example link.

		html = "<div>An<br/>example<br/>link.</div>";
		doc = Jsoup.parse(html);
		text = doc.select("div").first().wholeText();
		System.out.println(text);//An example link.
	}

	@Test
	public void testElementTextNodes() {
		/**
		 * textNodes
		 */
		String html = "<p>An<a href='http://example.com/'><b>example</b></a>link.</p>";
		Document doc = Jsoup.parse(html);
		List<TextNode> textNodes = doc.select("p").first().textNodes();
		System.out.println(textNodes);//[An, link.]
	}
	@Test
	public void test1() {
		/**
		 * 只包含自己的文本标签
		 */
		String html = "<p>One <span>Two</span> Three <br> Four</p>";
		Document doc = Jsoup.parse(html);
		Element element = doc.select("p").first();
		System.out.println(element.text());//"One Two Three Four"
		System.out.println(element.ownText());//One Three Four"
		System.out.println(element.children());//Elements[<span>, <br>]
		System.out.println(element.childNodes());//List<Node>["One ", <span>, " Three ", <br>, " Four"]
		System.out.println(element.textNodes());//List<TextNode>["One ", " Three ", " Four"]
	}


}
```

Elements方法与Element的关系：
```java
package com;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;
import org.junit.Test;

import java.util.List;

public class ElementsTest {
    private String html = "<html><head><title>The Dormouse's story</title></head>\n" +
            "<body>\n" +
            "<p class=\"title\"><b>The Dormouse's story</b></p>\n" +
            "\n" +
            "<p class=\"story\">Once upon a time there were three little sisters; and their names were\n" +
            "<a href=\"http://example.com/elsie\" class=\"sister\" id=\"link1\">Elsie</a>,\n" +
            "<a href=\"http://example.com/lacie\" class=\"sister\" id=\"link2\">Lacie</a> and\n" +
            "<a href=\"http://example.com/tillie\" class=\"sister\" id=\"link3\">Tillie</a>;\n" +
            "and they lived at the bottom of a well.</p>\n" +
            "\n" +
            "<p class=\"story\">...</p>";

    @Test
    public void test(){
        /**
         * Elements的prev、next、prevAll、nextAll都是分别对每个元素查找，
         * 字符串选择器只有next、nextAll的语法，函数的方式更多，而且速度更快，这点和jQuery一样
         *  prev、next、prevAll、nextAll 都有一个可选的参数用于过滤
         */
        Document doc = Jsoup.parse(html);
        //Get the immediate previous element sibling of each element in this list.
        Elements elements = doc.select("p").prev();
        System.out.println(elements.size());

        //Get the immediate previous element sibling of each element in this list, filtered by the query.
        elements = doc.select("p").prev("[class=title]");
        System.out.println(elements.size());

        //Get all of the previous element siblings of each element in this list.
        elements = doc.select("p").prevAll();
        System.out.println(elements.size());

        //Get the immediate next element sibling of each element in this list.
        elements = doc.select("p").next();
        System.out.println(elements.size());
    }

    @Test
    public void test1(){
        /**
         * String text()连接每个Element的element.text()，以空格分隔
         * List<String> eachText() 每个Element的element.text()放入集合，过滤空元素
         *
         * 看到这里我觉得Elements除了select方法，别的方法不看也行，Elements就是Element的list，详细看看Element的方法即可
         * Elements的操作我们可以自己实现，而且用java8甚至scala的语法更加灵活。
         */
        Document doc = Jsoup.parse(html);
        String text = doc.select("p").text();
        System.out.println(text);
        List<String> texts = doc.select("p").eachText();
        System.out.println(texts);
    }

    @Test
    public void test2(){
        /***
         * nextElementSibling()返回后一个同级元素
         * previousElementSibling()返回前一个同级元素
         * siblingElements()返回所有的同级元素（父元素的所有子元素），除了元素自己
         * firstElementSibling() 父元素的所有子元素的第一个
         * lastElementSibling() 父元素的所有子元素的最后一个
         * nextSibling、previousSibling这些方法名中不带Element的返回的是Node
         */
        Document doc = Jsoup.parse(html);
        Element element = doc.select("p").first();
        element.nextElementSibling();
        element.nextSibling();
        element.previousElementSibling();
        element.previousSibling();
        element.siblingElements();
        element.firstElementSibling();
        element.lastElementSibling();
    }

    @Test
    public void test3(){
        /**
         * 看一下Node的继承结构
         * Node有两个直接的子类：class Element、abstract class LeafNode
         * Element：A HTML element consists of a tag name, attributes, and child nodes (including text nodes and other elements).
         * LeafNode主要的子类有TextNode、Comment、DataNode。基本上都是建明之一。
         * DataNode：A data node, for contents of style, script tags etc, where contents should not show in text()
         */
        Document doc = Jsoup.parse(html);
        Element element = doc.select("p").first();
        element.dataNodes();
    }


}
```
