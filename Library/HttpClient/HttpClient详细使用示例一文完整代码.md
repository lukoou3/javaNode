# HttpClient详细使用示例一文完整代码

## pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.aspire</groupId>
	<artifactId>HttpClientDemo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.2.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		
		<!-- fastjson -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.47</version>
		</dependency>
		
		<!-- httpclient -->
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>4.5.5</version>
		</dependency>

		<!--
		     如果需要灵活的传输文件，引入次依赖后会更加方便
		-->
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpmime</artifactId>
			<version>4.5.5</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```
## application.yml
```yml
# 关于如何生成、配置证书，可参考本人的这篇博客:https://blog.csdn.net/justry_deng/article/details/91569132
http:
  port: 12345
server:
  port: 8484
  ssl:
    key-store: classpath:server/dsstore.p12
    key-store-password: dengshuai@123*Ds
    keyAlias: dengshuai_ca
    keyStoreType: PKCS12
```

## AbcHttpClientDemoApplication.java
```java
package com;

import org.apache.catalina.connector.Connector;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.servlet.MultipartConfigFactory;
import org.springframework.boot.web.servlet.server.ServletWebServerFactory;
import org.springframework.context.annotation.Bean;

import javax.servlet.MultipartConfigElement;

/**
 * 启动类
 *
 * @author JustryDeng
 * @date 2019/9/18 17:52
 */
@SpringBootApplication
public class AbcHttpClientDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(AbcHttpClientDemoApplication.class, args);
	}

	@Value("${http.port}")
	private Integer httpPort;

	@Bean
	public ServletWebServerFactory servletContainer() {
		TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
		tomcat.addAdditionalTomcatConnectors(createHttpConnector());
		return tomcat;
	}

	private Connector createHttpConnector() {
		Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
		connector.setScheme("http");
		connector.setSecure(false);
		connector.setPort(httpPort);
		return connector;
	}



	/**
	 * 对上传文件的配置
	 *
	 * @return MultipartConfigElement配置实例
	 * @date 2018年6月29日 上午10:55:02
	 */
	@Bean
	public MultipartConfigElement multipartConfig() {
		MultipartConfigFactory factory = new MultipartConfigFactory();
		// 设置单个附件大小上限值(默认为1M)
		//选择字符串作为参数的话，单位可以用MB、KB;
		factory.setMaxFileSize("200MB");
		// 设置所有附件的总大小上限值
		factory.setMaxRequestSize("1024MB");
		return factory.createMultipartConfig();
	}

}
```

## User.java
```java
package com.aspire.model;

/**
 * 用户实体类模型
 *
 * @author JustryDeng
 * @date 2018年7月13日 下午5:27:58
 */
public class User {

	/** 姓名 */
	private String name;

	/** 年龄 */
	private Integer age;

	/** 性别 */
	private String gender;

	/** 座右铭 */
	private String motto;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Integer getAge() {
		return age;
	}

	public void setAge(Integer age) {
		this.age = age;
	}

	public String getGender() {
		return gender;
	}

	public void setGender(String gender) {
		this.gender = gender;
	}

	public String getMotto() {
		return motto;
	}

	public void setMotto(String motto) {
		this.motto = motto;
	}

	@Override
	public String toString() {
		return age + "岁" + gender + "人[" + name + "]的座右铭居然是: " + motto + "!!!";
	}

}
```


## HttpClientController.java
```java
package com.aspire.controller;

import com.aspire.model.User;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.*;
import java.net.URLDecoder;
import java.nio.charset.StandardCharsets;
import java.util.List;

/**
 * HttpClient测试
 *
 * @author JustryDeng
 * @date 2018年7月13日 下午4:05:57
 */
@RestController
public class HttpClientController {

	/**
	 * GET无参
	 *
	 * @return 测试数据
	 * @date 2018年7月13日 下午5:29:44
	 */
	@RequestMapping("/doGetControllerOne")
	public String doGetControllerOne() {
		return "123";
	}

	/**
	 * GET有参
	 *
	 * @param name
	 *            姓名
	 * @param age
	 *            年龄
	 * @return 测试数据
	 * @date 2018年7月13日 下午5:29:47
	 */
	@RequestMapping("/doGetControllerTwo")
	public String doGetControllerTwo(String name, Integer age) {
		return "没想到[" + name + "]都" + age + "岁了!";
	}

	/**
	 * POST无参
	 *
	 * @return 测试数据
	 * @date 2018年7月13日 下午5:29:49
	 */
	@RequestMapping(value = "/doPostControllerOne", method = RequestMethod.POST)
	public String doPostControllerOne() {
		return "这个post请求没有任何参数!";
	}

	/**
	 * POST有参(对象参数)
	 *
	 * @return 测试数据
	 * @date 2018年7月13日 下午5:29:52
	 */
	@RequestMapping(value = "/doPostControllerTwo", method = RequestMethod.POST)
	public String doPostControllerTwo(@RequestBody User user) {
		return user.toString();
	}

	/**
	 * POST有参(普通参数 + 对象参数)
	 *
	 * @return 测试数据
	 * @date 2018年7月13日 下午5:29:52
	 */
	@RequestMapping(value = "/doPostControllerThree", method = RequestMethod.POST)
	public String doPostControllerThree(@RequestBody User user,Integer flag, String meaning) {
		return user.toString() + "\n" + flag + ">>>" + meaning;
	}
	
	/**
	 * POST有参(普通参数)
	 *
	 * @return 测试数据
	 * @date 2018年7月14日 上午10:54:29
	 */
	@RequestMapping(value = "/doPostControllerFour", method = RequestMethod.POST)
	public String doPostControllerThree1(String name, Integer age) {
		return "[" + name + "]居然才[" + age + "]岁!!!";
	}

	/**
	 * application/x-www-form-urlencoded 请求测试
	 *
	 * @date 2019/9/19 9:59
	 */
	@PostMapping(value = "/form/data")
	public String formDataControllerTest(@RequestParam("name") String name,
									  @RequestParam("age") Integer age) {
		return "application/x-www-form-urlencoded请求成功，name值为【" + name + "】，age值为【" + age + "】";
	}

	/**
	 * httpclient传文件测试
	 *
	 * 注: 即multipart/form-data测试。
	 * 注:多文件的话，可以使用数组MultipartFile[]或集合List<MultipartFile>来接收
	 * 注:单文件的话，可以直接使用MultipartFile来接收
	 *
	 * @date 2019/9/19 9:59
	 */
	@PostMapping(value = "/file")
	public String fileControllerTest(
			                         @RequestParam("name") String name,
									 @RequestParam("age") Integer age,
									 @RequestParam("files") List<MultipartFile> multipartFiles)
			                          throws UnsupportedEncodingException {

		StringBuilder sb = new StringBuilder(64);
		// 防止中文乱码
		sb.append("\n");
		sb.append("name=").append(name)
		  .append("\tage=").append(age);
		String fileName;
		for (MultipartFile file : multipartFiles) {
			sb.append("\n文件信息:\n");
			fileName = file.getOriginalFilename();
			if (fileName == null) {
				continue;
			}
			// 防止中文乱码
			// 在传文件时，将文件名URLEncode，然后在这里获取文件名时，URLDecode。就能避免乱码问题。
			fileName = URLDecoder.decode(fileName, "utf-8");
			sb.append("\t文件名: ").append(fileName);
			sb.append("\t文件大小: ").append(file.getSize() * 1.0 / 1024).append("KB");
			sb.append("\tContentType: ").append(file.getContentType());
			sb.append("\n");
		}
		return  sb.toString();
	}

	/**
	 * httpclient传流测试
	 *
	 * @date 2019/9/19 9:59
	 */
	@PostMapping(value = "/is")
	public String fileControllerTest(@RequestParam("name") String name, InputStream is) throws IOException {
		StringBuilder sb = new StringBuilder(64);
		sb.append("\nname值为: ").append(name);
		sb.append("\n输入流数据内容为: ");
		BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(is, StandardCharsets.UTF_8));
		String line;
		while ((line = bufferedReader.readLine()) != null) {
			sb.append(line);
		}
		return  sb.toString();
	}
}
```

## HttpClientDemoTest.java
```java
package com.test;

import com.AbcHttpClientDemoApplication;
import com.alibaba.fastjson.JSON;
import com.aspire.model.User;
import org.apache.http.HttpEntity;
import org.apache.http.NameValuePair;
import org.apache.http.ParseException;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.URI;
import java.net.URISyntaxException;
import java.net.URLEncoder;
import java.util.ArrayList;
import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = { AbcHttpClientDemoApplication.class })
public class HttpClientDemoTest {
	/**
	 * GET---无参测试
	 *
	 * @date 2018年7月13日 下午4:18:50
	 */
	@Test
	public void doGetTestOne() {
		// 获得Http客户端(可以理解为:你得先有一个浏览器;注意:实际上HttpClient与浏览器是不一样的)
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();
		// 创建Get请求
		HttpGet httpGet = new HttpGet("http://localhost:12345/doGetControllerOne");

		// 响应模型
		CloseableHttpResponse response = null;
		try {
			// 由客户端执行(发送)Get请求
			response = httpClient.execute(httpGet);
			// 从响应模型中获取响应实体
			HttpEntity responseEntity = response.getEntity();
			System.out.println("响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("响应内容长度为:" + responseEntity.getContentLength());
				System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * GET---有参测试 (方式一:手动在url后面加上参数)
	 *
	 * @date 2018年7月13日 下午4:19:23
	 */
	@Test
	public void doGetTestWayOne() {
		// 获得Http客户端(可以理解为:你得先有一个浏览器;注意:实际上HttpClient与浏览器是不一样的)
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();

		// 参数
		StringBuilder params = new StringBuilder();
		try {
			// 字符数据最好encoding以下;这样一来，某些特殊字符才能传过去(如:某人的名字就是“&”,不encoding的话,传不过去)
			params.append("name=").append(URLEncoder.encode("&", "utf-8"));
			params.append("&");
			params.append("age=24");
		} catch (UnsupportedEncodingException e1) {
			e1.printStackTrace();
		}

		// 创建Get请求
		HttpGet httpGet = new HttpGet("http://localhost:12345/doGetControllerTwo" + "?" + params);
		// 响应模型
		CloseableHttpResponse response = null;
		try {
			// 配置信息
			RequestConfig requestConfig = RequestConfig.custom()
					// 设置连接超时时间(单位毫秒)
					.setConnectTimeout(5000)
					// 设置请求超时时间(单位毫秒)
					.setConnectionRequestTimeout(5000)
					// socket读写超时时间(单位毫秒)
					.setSocketTimeout(5000)
					// 设置是否允许重定向(默认为true)
					.setRedirectsEnabled(true).build();

			// 将上面的配置信息 运用到这个Get请求里
			httpGet.setConfig(requestConfig);

			// 由客户端执行(发送)Get请求
			response = httpClient.execute(httpGet);
			// 从响应模型中获取响应实体
			HttpEntity responseEntity = response.getEntity();
			System.out.println("响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("响应内容长度为:" + responseEntity.getContentLength());
				System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * GET---有参测试 (方式二:将参数放入键值对类中,再放入URI中,从而通过URI得到HttpGet实例)
	 *
	 * @date 2018年7月13日 下午4:19:23
	 */
	@Test
	public void doGetTestWayTwo() {
		// 获得Http客户端(可以理解为:你得先有一个浏览器;注意:实际上HttpClient与浏览器是不一样的)
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();

		// 参数
		URI uri = null;
		try {
			// 将参数放入键值对类NameValuePair中,再放入集合中
			List<NameValuePair> params = new ArrayList<>();
			params.add(new BasicNameValuePair("name", "&"));
			params.add(new BasicNameValuePair("age", "18"));
			// 设置uri信息,并将参数集合放入uri;
			// 注:这里也支持一个键值对一个键值对地往里面放setParameter(String key, String value)
			uri = new URIBuilder().setScheme("http").setHost("localhost")
					              .setPort(12345).setPath("/doGetControllerTwo")
					              .setParameters(params).build();
		} catch (URISyntaxException e1) {
			e1.printStackTrace();
		}
		// 创建Get请求
		HttpGet httpGet = new HttpGet(uri);

		// 响应模型
		CloseableHttpResponse response = null;
		try {
			// 配置信息
			RequestConfig requestConfig = RequestConfig.custom()
					// 设置连接超时时间(单位毫秒)
					.setConnectTimeout(5000)
					// 设置请求超时时间(单位毫秒)
					.setConnectionRequestTimeout(5000)
					// socket读写超时时间(单位毫秒)
					.setSocketTimeout(5000)
					// 设置是否允许重定向(默认为true)
					.setRedirectsEnabled(true).build();

			// 将上面的配置信息 运用到这个Get请求里
			httpGet.setConfig(requestConfig);

			// 由客户端执行(发送)Get请求
			response = httpClient.execute(httpGet);
			// 从响应模型中获取响应实体
			HttpEntity responseEntity = response.getEntity();
			System.out.println("响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("响应内容长度为:" + responseEntity.getContentLength());
				System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * POST---无参测试
	 *
	 * @date 2018年7月13日 下午4:18:50
	 */
	@Test
	public void doPostTestOne() {

		// 获得Http客户端(可以理解为:你得先有一个浏览器;注意:实际上HttpClient与浏览器是不一样的)
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();

		// 创建Post请求
		HttpPost httpPost = new HttpPost("http://localhost:12345/doPostControllerOne");
		// 响应模型
		CloseableHttpResponse response = null;
		try {
			// 由客户端执行(发送)Post请求
			response = httpClient.execute(httpPost);
			// 从响应模型中获取响应实体
			HttpEntity responseEntity = response.getEntity();

			System.out.println("响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("响应内容长度为:" + responseEntity.getContentLength());
				System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * POST---有参测试(对象参数)
	 *
	 * @date 2018年7月13日 下午4:18:50
	 */
	@Test
	public void doPostTestTwo() {

		// 获得Http客户端(可以理解为:你得先有一个浏览器;注意:实际上HttpClient与浏览器是不一样的)
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();

		// 创建Post请求
		HttpPost httpPost = new HttpPost("http://localhost:12345/doPostControllerTwo");
		User user = new User();
		user.setName("潘晓婷");
		user.setAge(18);
		user.setGender("女");
		user.setMotto("姿势要优雅~");
		// 我这里利用阿里的fastjson，将Object转换为json字符串;
		// (需要导入com.alibaba.fastjson.JSON包)
		String jsonString = JSON.toJSONString(user);


		StringEntity entity = new StringEntity(jsonString, "UTF-8");

		// post请求是将参数放在请求体里面传过去的;这里将entity放入post请求体中
		httpPost.setEntity(entity);

		httpPost.setHeader("Content-Type", "application/json;charset=utf8");

		// 响应模型
		CloseableHttpResponse response = null;
		try {
			// 由客户端执行(发送)Post请求
			response = httpClient.execute(httpPost);
			// 从响应模型中获取响应实体
			HttpEntity responseEntity = response.getEntity();

			System.out.println("响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("响应内容长度为:" + responseEntity.getContentLength());
				System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * POST---有参测试(基本参数 + 对象参数)
	 *
	 * @date 2018年7月13日 下午4:18:50
	 */
	@Test
	public void doPostTestThree() {

		// 获得Http客户端(可以理解为:你得先有一个浏览器;注意:实际上HttpClient与浏览器是不一样的)
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();

		// 创建Post请求
		// 参数
		URI uri = null;
		try {
			// 将参数放入键值对类NameValuePair中,再放入集合中
			List<NameValuePair> params = new ArrayList<>();
			params.add(new BasicNameValuePair("flag", "4"));
			params.add(new BasicNameValuePair("meaning", "这是什么鬼？"));
			// 设置uri信息,并将参数集合放入uri;
			// 注:这里也支持一个键值对一个键值对地往里面放setParameter(String key, String value)
			uri = new URIBuilder().setScheme("http").setHost("localhost").setPort(12345)
					.setPath("/doPostControllerThree").setParameters(params).build();
		} catch (URISyntaxException e1) {
			e1.printStackTrace();
		}

		HttpPost httpPost = new HttpPost(uri);
		// HttpPost httpPost = new
		// HttpPost("http://localhost:12345/doPostControllerThree1");

		// 创建user参数
		User user = new User();
		user.setName("潘晓婷");
		user.setAge(18);
		user.setGender("女");
		user.setMotto("姿势要优雅~");

		// 将user对象转换为json字符串，并放入entity中
		StringEntity entity = new StringEntity(JSON.toJSONString(user), "UTF-8");

		// post请求是将参数放在请求体里面传过去的;这里将entity放入post请求体中
		httpPost.setEntity(entity);

		httpPost.setHeader("Content-Type", "application/json;charset=utf8");

		// 响应模型
		CloseableHttpResponse response = null;
		try {
			// 由客户端执行(发送)Post请求
			response = httpClient.execute(httpPost);
			// 从响应模型中获取响应实体
			HttpEntity responseEntity = response.getEntity();

			System.out.println("响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("响应内容长度为:" + responseEntity.getContentLength());
				System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * POST---有参测试(基本参数)
	 *
	 * @date 2018年7月13日 下午4:18:50
	 */
	@Test
	public void doPostTestFour() {

		// 获得Http客户端(可以理解为:你得先有一个浏览器;注意:实际上HttpClient与浏览器是不一样的)
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();

		// 参数
		StringBuilder params = new StringBuilder();
		try {
			// 字符数据最好encoding以下;这样一来，某些特殊字符才能传过去(如:某人的名字就是“&”,不encoding的话,传不过去)
			params.append("name=").append(URLEncoder.encode("&", "utf-8"));
			params.append("&");
			params.append("age=24");
		} catch (UnsupportedEncodingException e1) {
			e1.printStackTrace();
		}

		// 创建Post请求
		HttpPost httpPost = new HttpPost("http://localhost:12345/doPostControllerFour" + "?" + params);

		// 设置ContentType(注:如果只是传普通参数的话,ContentType不一定非要用application/json)
		httpPost.setHeader("Content-Type", "application/json;charset=utf8");

		// 响应模型
		CloseableHttpResponse response = null;
		try {
			// 由客户端执行(发送)Post请求
			response = httpClient.execute(httpPost);
			// 从响应模型中获取响应实体
			HttpEntity responseEntity = response.getEntity();

			System.out.println("响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("响应内容长度为:" + responseEntity.getContentLength());
				System.out.println("响应内容为:" + EntityUtils.toString(responseEntity));
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
}
```

## SupplementTest.java
```java
package com.test;

import com.AbcHttpClientDemoApplication;
import org.apache.http.HttpEntity;
import org.apache.http.ParseException;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.FileEntity;
import org.apache.http.entity.InputStreamEntity;
import org.apache.http.entity.StringEntity;
import org.apache.http.entity.mime.FormBodyPart;
import org.apache.http.entity.mime.FormBodyPartBuilder;
import org.apache.http.entity.mime.MultipartEntityBuilder;
import org.apache.http.entity.mime.content.FileBody;
import org.apache.http.entity.mime.content.StringBody;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.codec.multipart.FilePart;
import org.springframework.test.context.junit4.SpringRunner;

import javax.net.ssl.SSLContext;
import javax.net.ssl.TrustManager;
import javax.net.ssl.TrustManagerFactory;
import javax.net.ssl.X509TrustManager;
import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.net.URLEncoder;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.security.*;
import java.security.cert.CertificateException;
import java.security.cert.CertificateFactory;
import java.security.cert.X509Certificate;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * 知识补充
 *
 * @date 2019年9月18日
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = { AbcHttpClientDemoApplication.class })
public class SupplementTest {


	/**
	 * 防止响应乱码
	 */
	@Test
	public void test1() {
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();
		HttpGet httpGet = new HttpGet("http://localhost:12345/doGetControllerOne");
		CloseableHttpResponse response = null;
		try {
			response = httpClient.execute(httpGet);
			HttpEntity responseEntity = response.getEntity();
			System.out.println("响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("响应内容长度为:" + responseEntity.getContentLength());
				// 主动设置编码，来防止响应乱码
				String responseStr = EntityUtils.toString(responseEntity, StandardCharsets.UTF_8);
				System.out.println("响应内容为:" + responseStr);
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/// ------------------------------------------------------------------------ 分割线
	
	/**
	 * 发送HTTPS请求
	 */
	@Test
	public void test2() {
		CloseableHttpClient httpClient = getHttpClient(true);
		HttpGet httpGet = new HttpGet("https://localhost:8484/doGetControllerOne");
		CloseableHttpResponse response = null;
		try {
			response = httpClient.execute(httpGet);
			HttpEntity responseEntity = response.getEntity();
			System.out.println("HTTPS响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("HTTPS响应内容长度为:" + responseEntity.getContentLength());
				// 主动设置编码，来防止响应乱码
				String responseStr = EntityUtils.toString(responseEntity, StandardCharsets.UTF_8);
				System.out.println("HTTPS响应内容为:" + responseStr);
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * 根据是否是https请求，获取HttpClient客户端
	 *
	 * TODO 本人这里没有进行完美封装。对于 校不校验校验证书的选择，本人这里是写死
	 *      在代码里面的，你们再使用时，可以灵活二次封装。
	 *
	 * 提示: 此工具类的封装、相关客户端、服务端证书的生成，可参考我的这篇博客:
	 *      <linked>https://blog.csdn.net/justry_deng/article/details/91569132</linked>
	 *
	 *
	 * @param isHttps 是否是HTTPS请求
	 *
	 * @return  HttpClient实例
	 * @date 2019/9/18 17:57
	 */
	private CloseableHttpClient getHttpClient(boolean isHttps) {
		CloseableHttpClient httpClient;
		if (isHttps) {
			SSLConnectionSocketFactory sslSocketFactory;
			try {
				/// 如果不作证书校验的话
				sslSocketFactory = getSocketFactory(false, null, null);

				/// 如果需要证书检验的话
				// 证书
				//InputStream ca = this.getClass().getClassLoader().getResourceAsStream("client/ds.crt");
				// 证书的别名，即:key。 注:cAalias只需要保证唯一即可，不过推荐使用生成keystore时使用的别名。
				// String cAalias = System.currentTimeMillis() + "" + new SecureRandom().nextInt(1000);
				//sslSocketFactory = getSocketFactory(true, ca, cAalias);
			} catch (Exception e) {
				throw new RuntimeException(e);
			}
			httpClient = HttpClientBuilder.create().setSSLSocketFactory(sslSocketFactory).build();
			return httpClient;
		}
		httpClient = HttpClientBuilder.create().build();
		return httpClient;
	}

	/**
	 * HTTPS辅助方法, 为HTTPS请求 创建SSLSocketFactory实例、TrustManager实例
	 *
	 * @param needVerifyCa
	 *         是否需要检验CA证书(即:是否需要检验服务器的身份)
	 * @param caInputStream
	 *         CA证书。(若不需要检验证书，那么此处传null即可)
	 * @param cAalias
	 *         别名。(若不需要检验证书，那么此处传null即可)
	 *         注意:别名应该是唯一的， 别名不要和其他的别名一样，否者会覆盖之前的相同别名的证书信息。别名即key-value中的key。
	 *
	 * @return SSLConnectionSocketFactory实例
	 * @throws NoSuchAlgorithmException
	 *         异常信息
	 * @throws CertificateException
	 *         异常信息
	 * @throws KeyStoreException
	 *         异常信息
	 * @throws IOException
	 *         异常信息
	 * @throws KeyManagementException
	 *         异常信息
	 * @date 2019/6/11 19:52
	 */
	private static SSLConnectionSocketFactory getSocketFactory(boolean needVerifyCa, InputStream caInputStream, String cAalias)
			throws CertificateException, NoSuchAlgorithmException, KeyStoreException,
			IOException, KeyManagementException {
		X509TrustManager x509TrustManager;

		// https请求，需要校验证书
		if (needVerifyCa) {
			KeyStore keyStore = getKeyStore(caInputStream, cAalias);
			TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
			trustManagerFactory.init(keyStore);
			TrustManager[] trustManagers = trustManagerFactory.getTrustManagers();
			if (trustManagers.length != 1 || !(trustManagers[0] instanceof X509TrustManager)) {
				throw new IllegalStateException("Unexpected default trust managers:" + Arrays.toString(trustManagers));
			}
			x509TrustManager = (X509TrustManager) trustManagers[0];
			// 这里传TLS或SSL其实都可以的
			SSLContext sslContext = SSLContext.getInstance("TLS");
			sslContext.init(null, new TrustManager[]{x509TrustManager}, new SecureRandom());
			return new SSLConnectionSocketFactory(sslContext);
		}
		// https请求，不作证书校验
		x509TrustManager = new X509TrustManager() {
			@Override
			public void checkClientTrusted(X509Certificate[] arg0, String arg1) {
			}

			@Override
			public void checkServerTrusted(X509Certificate[] arg0, String arg1) {
				// 不验证
			}

			@Override
			public X509Certificate[] getAcceptedIssuers() {
				return new X509Certificate[0];
			}
		};
		SSLContext sslContext = SSLContext.getInstance("TLS");
		sslContext.init(null, new TrustManager[]{x509TrustManager}, new SecureRandom());
		return new SSLConnectionSocketFactory(sslContext);
	}

	/**
	 * 获取(密钥及证书)仓库
	 * 注:该仓库用于存放 密钥以及证书
	 *
	 * @param caInputStream
	 *         CA证书(此证书应由要访问的服务端提供)
	 * @param cAalias
	 *         别名
	 *         注意:别名应该是唯一的， 别名不要和其他的别名一样，否者会覆盖之前的相同别名的证书信息。别名即key-value中的key。
	 * @return 密钥、证书 仓库
	 * @throws KeyStoreException 异常信息
	 * @throws CertificateException 异常信息
	 * @throws IOException 异常信息
	 * @throws NoSuchAlgorithmException 异常信息
	 * @date 2019/6/11 18:48
	 */
	private static KeyStore getKeyStore(InputStream caInputStream, String cAalias)
			throws KeyStoreException, CertificateException, IOException, NoSuchAlgorithmException {
		// 证书工厂
		CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
		// 秘钥仓库
		KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
		keyStore.load(null);
		keyStore.setCertificateEntry(cAalias, certificateFactory.generateCertificate(caInputStream));
		return keyStore;
	}

	/// ------------------------------------------------------------------------ 分割线

	/**
	 * application/x-www-form-urlencoded 请求(即:表单请求，表单数据编码为键值对)
	 *
	 * 注:multipart/form-data也属于表单请求。不过其将表单数据编码为了一条消息，每个控件对应消息的一部分。
	 *
	 * 注:没有文件的话，用默认的application/x-www-form-urlencoded就行;
	 *    有文件的话，要用multipart/form-data进行编码编码。
	 *
	 */
	@Test
	public void test3() {
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();
		HttpPost httpPost = new HttpPost("http://localhost:12345/form/data");
		CloseableHttpResponse response = null;
		try {
			httpPost.setHeader("Content-Type", "application/x-www-form-urlencoded");

			List<BasicNameValuePair> params = new ArrayList<>(2);
			params.add(new BasicNameValuePair("name", "邓沙利文"));
			params.add(new BasicNameValuePair("age", "25"));
			UrlEncodedFormEntity formEntity = new UrlEncodedFormEntity(params, StandardCharsets.UTF_8);
			httpPost.setEntity(formEntity);

			response = httpClient.execute(httpPost);
			HttpEntity responseEntity = response.getEntity();
			System.out.println("HTTPS响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("HTTPS响应内容长度为:" + responseEntity.getContentLength());
				// 主动设置编码，来防止响应乱码
				String responseStr = EntityUtils.toString(responseEntity, StandardCharsets.UTF_8);
				System.out.println("HTTPS响应内容为:" + responseStr);
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/// ------------------------------------------------------------------------ 分割线

	/**
	 *
	 * 发送文件
	 *
	 * multipart/form-data传递文件(及相关信息)
	 *
	 * 注:如果想要灵活方便的传输文件的话，
	 *    除了引入org.apache.httpcomponents基本的httpclient依赖外
	 *    在额外引入org.apache.httpcomponents的httpmime依赖。
	 *    追注:即便不引入httpmime依赖，也是能传输文件的，不过功能不够强大。
	 *
	 */
	@Test
	public void test4() {
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();
		HttpPost httpPost = new HttpPost("http://localhost:12345/file");
		CloseableHttpResponse response = null;
		try {
			MultipartEntityBuilder multipartEntityBuilder = MultipartEntityBuilder.create();
			// 第一个文件
			String filesKey = "files";
			File file1 = new File("C:\\Users\\JustryDeng\\Desktop\\back.jpg");
			multipartEntityBuilder.addBinaryBody(filesKey, file1);
			// 第二个文件(多个文件的话，使用可一个key就行，后端用数组或集合进行接收即可)
			File file2 = new File("C:\\Users\\JustryDeng\\Desktop\\头像.jpg");
			// 防止服务端收到的文件名乱码。 我们这里可以先将文件名URLEncode，然后服务端拿到文件名时在URLDecode。就能避免乱码问题。
			// 文件名其实是放在请求头的Content-Disposition里面进行传输的，如其值为form-data; name="files"; filename="头像.jpg"
			multipartEntityBuilder.addBinaryBody(filesKey, file2, ContentType.DEFAULT_BINARY, URLEncoder.encode(file2.getName(), "utf-8"));
			// 其它参数(注:自定义contentType，设置UTF-8是为了防止服务端拿到的参数出现乱码)
			ContentType contentType = ContentType.create("text/plain", Charset.forName("UTF-8"));
			multipartEntityBuilder.addTextBody("name", "邓沙利文", contentType);
			multipartEntityBuilder.addTextBody("age", "25", contentType);

			HttpEntity httpEntity = multipartEntityBuilder.build();
			httpPost.setEntity(httpEntity);

			response = httpClient.execute(httpPost);
			HttpEntity responseEntity = response.getEntity();
			System.out.println("HTTPS响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("HTTPS响应内容长度为:" + responseEntity.getContentLength());
				// 主动设置编码，来防止响应乱码
				String responseStr = EntityUtils.toString(responseEntity, StandardCharsets.UTF_8);
				System.out.println("HTTPS响应内容为:" + responseStr);
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	/// ------------------------------------------------------------------------ 分割线

	/**
	 *
	 * 发送流
	 *
	 */
	@Test
	public void test5() {
		CloseableHttpClient httpClient = HttpClientBuilder.create().build();
		HttpPost httpPost = new HttpPost("http://localhost:12345/is?name=邓沙利文");
		CloseableHttpResponse response = null;
		try {
			InputStream is = new ByteArrayInputStream("流啊流~".getBytes());
			InputStreamEntity ise = new InputStreamEntity(is);
			httpPost.setEntity(ise);

			response = httpClient.execute(httpPost);
			HttpEntity responseEntity = response.getEntity();
			System.out.println("HTTPS响应状态为:" + response.getStatusLine());
			if (responseEntity != null) {
				System.out.println("HTTPS响应内容长度为:" + responseEntity.getContentLength());
				// 主动设置编码，来防止响应乱码
				String responseStr = EntityUtils.toString(responseEntity, StandardCharsets.UTF_8);
				System.out.println("HTTPS响应内容为:" + responseStr);
			}
		} catch (ParseException | IOException e) {
			e.printStackTrace();
		} finally {
			try {
				// 释放资源
				if (httpClient != null) {
					httpClient.close();
				}
				if (response != null) {
					response.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
}
```





```java

```
