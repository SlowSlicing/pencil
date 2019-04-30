Http Client 4.5.2
head
&emsp;&emsp;我们平常浏览各个网站时，不免有时候就需要填写一些信息，比如注册时，登录时，这些信息一般都是通过GET请求或者POST`（敏感信息一般使用POST，数据隐藏，相对来说更安全)`请求提交到后台，经过后台的一系列处理，再返回给前台结果，前台进行处理。

---

### GET请求携带请求参数和请求头：

```
@Test
public void getParams() {
	
	// 获取连接客户端工具
	CloseableHttpClient httpClient = HttpClients.createDefault();
	
	String entityStr = null;
	CloseableHttpResponse response = null;
	
	try {
		/*
		 * 由于GET请求的参数都是拼装在URL地址后方，所以我们要构建一个URL，带参数
		 */
		URIBuilder uriBuilder = new URIBuilder("http://www.baidu.com");
		/** 第一种添加参数的形式 */
		/*uriBuilder.addParameter("name", "root");
		uriBuilder.addParameter("password", "123456");*/
		/** 第二种添加参数的形式 */
		List<NameValuePair> list = new LinkedList<>();
		BasicNameValuePair param1 = new BasicNameValuePair("name", "root");
		BasicNameValuePair param2 = new BasicNameValuePair("password", "123456");
		list.add(param1);
		list.add(param2);
		uriBuilder.setParameters(list);
		
		// 根据带参数的URI对象构建GET请求对象
		HttpGet httpGet = new HttpGet(uriBuilder.build());
		
		/* 
		 * 添加请求头信息
		 */
		// 浏览器表示
		httpGet.addHeader("User-Agent", "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)");
		// 传输的类型
		httpGet.addHeader("Content-Type", "application/x-www-form-urlencoded");
		
		// 执行请求
		response = httpClient.execute(httpGet);
		// 获得响应的实体对象
		HttpEntity entity = response.getEntity();
		// 使用Apache提供的工具类进行转换成字符串
		entityStr = EntityUtils.toString(entity, "UTF-8");
	} catch (ClientProtocolException e) {
		System.err.println("Http协议出现问题");
		e.printStackTrace();
	} catch (ParseException e) {
		System.err.println("解析错误");
		e.printStackTrace();
	} catch (URISyntaxException e) {
		System.err.println("URI解析异常");
		e.printStackTrace();
	} catch (IOException e) {
		System.err.println("IO异常");
		e.printStackTrace();
	} finally {
		// 释放连接
		if (null != response) {
			try {
				response.close();
				httpClient.close();
			} catch (IOException e) {
				System.err.println("释放连接出错");
				e.printStackTrace();
			}
		}
	}
	
	// 打印响应内容
	System.out.println(entityStr);
	
}
```

&emsp;&emsp;因为GET请求的参数都是拼装到URL后面进行传输的，所以这地方不能直接添加参数，需要组装好一个带参数的URI传递到HttpGet的构造方法中，构造一个带参数的GET请求。构造带参数的URI使用`URIBuilder`类。

&emsp;&emsp;上面添加请求参数的方法有两种，**`建议后者`**，后者操作更加灵活。

> * 打印结果：

![响应结果](http://img.lynchj.com/HttpClients/GET%E6%B7%BB%E5%8A%A0%E8%AF%B7%E6%B1%82%E5%8F%82%E6%95%B0%E5%92%8C%E8%AF%B7%E6%B1%82%E5%A4%B4%EF%BC%8C%E8%BF%94%E5%9B%9E%E6%AD%A3%E5%B8%B8.jpg '响应结果')

---

### POST请求携带请求参数和请求头：

```
@Test
public void postParams() {
	// 获取连接客户端工具
	CloseableHttpClient httpClient = HttpClients.createDefault();
	
	String entityStr = null;
	CloseableHttpResponse response = null;
	
	try {
		
		// 创建POST请求对象
		HttpPost httpPost = new HttpPost("http://www.baidu.com");
		
		/*
		 * 添加请求参数
		 */
		// 创建请求参数
		List<NameValuePair> list = new LinkedList<>();
		BasicNameValuePair param1 = new BasicNameValuePair("name", "root");
		BasicNameValuePair param2 = new BasicNameValuePair("password", "123456");
		list.add(param1);
		list.add(param2);
		// 使用URL实体转换工具
		UrlEncodedFormEntity entityParam = new UrlEncodedFormEntity(list, "UTF-8");
		httpPost.setEntity(entityParam);
		
		/* 
		 * 添加请求头信息
		 */
		// 浏览器表示
		httpPost.addHeader("User-Agent", "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)");
		// 传输的类型
		httpPost.addHeader("Content-Type", "application/x-www-form-urlencoded");
		
		// 执行请求
		response = httpClient.execute(httpPost);
		// 获得响应的实体对象
		HttpEntity entity = response.getEntity();
		// 使用Apache提供的工具类进行转换成字符串
		entityStr = EntityUtils.toString(entity, "UTF-8");
		
		// System.out.println(Arrays.toString(response.getAllHeaders()));
		
	} catch (ClientProtocolException e) {
		System.err.println("Http协议出现问题");
		e.printStackTrace();
	} catch (ParseException e) {
		System.err.println("解析错误");
		e.printStackTrace();
	} catch (IOException e) {
		System.err.println("IO异常");
		e.printStackTrace();
	} finally {
		// 释放连接
		if (null != response) {
			try {
				response.close();
				httpClient.close();
			} catch (IOException e) {
				System.err.println("释放连接出错");
				e.printStackTrace();
			}
		}
	}
	
	// 打印响应内容
	System.out.println(entityStr);
}
```

> * 打印结果：

![响应结果](http://img.lynchj.com/HttpClients/post302%E5%93%8D%E5%BA%94%E7%BB%93%E6%9E%9C.jpg '响应结果')
&emsp;&emsp;What？返回的为什么是个这？？？很显然，返回来的代码是`302`，**重定向**。

&emsp;&emsp;GET和POST两种请求方式，GET方式请求无这种情况产生，说明`HttpClient`对GET进行了*处理*，会`自动处理`重定向的连接。而`POST并没有`做这种处理，所以要`手动处理`。

&emsp;&emsp;PS：`这里有一个小注意事项，在POST请求参数转换成请求实体(Entity)时，注意编码格式问题：UrlEncodedFormEntity entityParam = new UrlEncodedFormEntity(list, "UTF-8");`
　　
&emsp;&emsp;我们该怎么解决这种问题？首先我们要分析，既然是重定向，那么响应回来的信息中，`头信息(hearder)`中肯定会存在重定向需要使用到的的`键值对（Location:"http://www.xxxx.com")`，先获取所有的头信息进行打印查看。

```
......前省略

	// 执行请求
	response = httpClient.execute(httpPost);
	// 获得响应的实体对象
	HttpEntity entity = response.getEntity();
	// 使用Apache提供的工具类进行转换成字符串
	entityStr = EntityUtils.toString(entity, "UTF-8");
	
	/** 此处获取所有的响应头信息并进行打印 */
	System.out.println(Arrays.toString(response.getAllHeaders()));
	
} catch (ClientProtocolException e) {
	System.err.println("Http协议出现问题");
	e.printStackTrace();
} catch (ParseException e) {

......后省略
```

> * 打印结果

![响应结果](http://img.lynchj.com/HttpClients/%E8%8E%B7%E5%8F%96%E6%89%80%E6%9C%89%E7%9A%84%E8%AF%B7%E6%B1%82%E5%A4%B4%E4%BF%A1%E6%81%AF.jpg '响应结果')

&emsp;&emsp;**`这时我们只需要获取到Location的值，再对其进行再次访问即可`**

&emsp;&emsp;其实这里隐含了一个问题，`敲重点...`这里我直接获取所有的响应信息的头信息并进行打印，其中是存在`Location`这个`key`的，这是因为在代码中具有以下代码所以才会有这个返回信息。

> * 浏览器标识请求头信息：

```
httpPost.addHeader("User-Agent", "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)");
```

如果没有上面的浏览器标识的话，有些网站是不会显示`Location`地址的。
~~httpPost.addHeader("User-Agent", "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)");~~
> * 打印结果：
[Server: bfe/1.0.8.18, Date: Tue, 18 Jul 2017 06:28:09 GMT, Content-Type: text/html, Content-Length: 17931, Connection: Keep-Alive, ETag: "54d9748e-460b"]

　　可以看到，其中并没有`Location`，这时候就需要加上`httpPost.addHeader("User-Agent", "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)");`。
