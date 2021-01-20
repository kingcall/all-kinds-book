##  一、HttpClient简介

　　HttpClient 是 Apache Jakarta Common 下的子项目，可以用来提供高效的、最新的、功能丰富的支持 HTTP 协议的客户端编程工具包，

　　并且它支持 HTTP 协议最新的版本和建议。

　　官方站点：http://hc.apache.org/　　　

　　最新版本4.5 http://hc.apache.org/httpcomponents-client-4.5.x/　

　　官方文档： http://hc.apache.org/httpcomponents-client-4.5.x/tutorial/html/index.html

　　Maven地址：

```
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.2</version>
</dependency>
```

　　HTTP 协议可能是现在 Internet 上使用得最多、最重要的协议了，越来越多的 Java 应用程序需要直接通过 HTTP 协议来访问网络资源。虽然在 JDK 的 java net包中

　　已经提供了访问 HTTP 协议的基本功能，但是对于大部分应用程序来说，JDK 库本身提供的功能还不够丰富和灵活。HttpClient 是 Apache Jakarta Common 下的子

　　项目，用来提供高效的、最新的、功能丰富的支持 HTTP 协议的客户端编程工具包，并且它支持 HTTP 协议最新的版本和建议。HttpClient 已经应用在很多的项目中，

　　比如 Apache Jakarta 上很著名的另外两个开源项目 Cactus 和 [HTMLUnit](http://baike.baidu.com/view/3724597.htm) 都使用了 HttpClient。现在HttpClient最新版本为 HttpClient 4.5 (GA) （2015-09-11）。

　　总结：我们搞爬虫的，主要是用HttpClient模拟浏览器请求第三方站点url，然后响应，获取网页数据，然后用Jsoup来提取我们需要的信息。

## 二、使用HttpClient获取网页内容

　　这里我们来抓取博客园首页的源码内容



```
package com.jxlg.study.httpclient;

import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.IOException;

public class GetWebPageContent {
    /**
     * 抓取网页信息使用get请求
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        //创建httpClient实例
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //创建httpGet实例
        HttpGet httpGet = new HttpGet("http://www.cnblogs.com");
        CloseableHttpResponse response = httpClient.execute(httpGet);
        if (response != null){
            HttpEntity entity =  response.getEntity();  //获取网页内容
            String result = EntityUtils.toString(entity, "UTF-8");
            System.out.println("网页内容:"+result);
        }
        if (response != null){
            response.close();
        }
        if (httpClient != null){
            httpClient.close();
        }
    }
}
```



　　 上述代码中可以直接获取到 网页内容，有的获取到的内容是 中文乱码的，这就需要根据 网页的编码 来设置编码了，比如gb2312。

## 三、模拟浏览器抓取网页

### 3.1、设置请求头消息User-Agent模拟浏览器

　　当我们使用上面写的那个代码去获取推酷的网页源码是（http://www.tuicool.com）时，会返回给我们如下信息：



```
网页内容:<!DOCTYPE html>
<html>
    <head>
          <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    </head>
    <body>
        <p>系统检测亲不是真人行为，因系统资源限制，我们只能拒绝你的请求。如果你有疑问，可以通过微博 http://weibo.com/tuicool2012/ 联系我们。</p>
    </body>
</html>
```



　　这是因为网站做了限制，限制别人爬。解决方式可以设置请求头消息 User-Agent模拟浏览器。代码如下：



```
import java.io.IOException;

public class GetWebPageContent {
    /**
     * 抓取网页信息使用get请求
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        //创建httpClient实例
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //创建httpGet实例
        HttpGet httpGet = new HttpGet("http://www.tuicool.com");
        httpGet.setHeader("User-Agent","Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36");
        CloseableHttpResponse response = httpClient.execute(httpGet);
        if (response != null){
            HttpEntity entity =  response.getEntity();  //获取网页内容
            String result = EntityUtils.toString(entity, "UTF-8");
            System.out.println("网页内容:"+result);
        }
        if (response != null){
            response.close();
        }
        if (httpClient != null){
            httpClient.close();
        }
    }
}
```



　　给HttpGet方法设置头消息，即可模拟浏览器访问。

### 3.2、获取响应内容Content-Type

　　使用  entity.getContentType().getValue()  来获取Content-Type，代码如下：



```
public class GetWebPageContent {
    /**
     * 抓取网页信息使用get请求
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        //创建httpClient实例
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //创建httpGet实例
        HttpGet httpGet = new HttpGet("http://www.tuicool.com");
        httpGet.setHeader("User-Agent","Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36");
        CloseableHttpResponse response = httpClient.execute(httpGet);
        if (response != null){
            HttpEntity entity =  response.getEntity();  //获取网页内容
            System.out.println("Content-Type："+entity.getContentType().getValue());  //获取Content-Type
        }
        if (response != null){
            response.close();
        }
        if (httpClient != null){
            httpClient.close();
        }
    }
}
```



　　结果：

　　　　![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/999804-20171214154338701-1150879337.png)

　　一般网页是text/html当然有些是带编码的，比如请求www.tuicool.com：输出：

　　　　Content-Type:text/html; charset=utf-8

　　假如请求js文件，比如 http://www.open1111.com/static/js/jQuery.js

　　运行输出：

　　　　Content-Type:application/javascript

　　假如请求的是文件，比如 http://central.maven.org/maven2/HTTPClient/HTTPClient/0.3-3/HTTPClient-0.3-3.jar

　　运行输出：

　　　　Content-Type:application/java-archive

　　当然Content-Type还有一堆，那这东西对于我们爬虫有啥用的，我们再爬取网页的时候 ，可以通过

　　Content-Type来提取我们需要爬取的网页或者是爬取的时候，需要过滤掉的一些网页。

### 3.3、获取响应状态

　　使用 response.getStatusLine().getStatusCode() 获取响应状态，代码如下：



```
public class GetWebPageContent {
    /**
     * 抓取网页信息使用get请求
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        //创建httpClient实例
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //创建httpGet实例
        HttpGet httpGet = new HttpGet("http://www.tuicool.com");
        httpGet.setHeader("User-Agent","Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36");
        CloseableHttpResponse response = httpClient.execute(httpGet);
        if (response != null){
            int statusCode = response.getStatusLine().getStatusCode();
            System.out.println("响应状态:"+statusCode);
        }
        if (response != null){
            response.close();
        }
        if (httpClient != null){
            httpClient.close();
        }
    }
} 
```



　　结果：

　　　　![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/999804-20171214155022295-1230789981.png)

　　我们HttpClient向服务器请求时，正常情况 执行成功 返回200状态码，不一定每次都会请求成功，

　　比如这个请求地址不存在 返回404，服务器内部报错， 返回500有些服务器有防采集，假如你频繁的采集数据，则返回403 拒绝你请求。

　　当然 我们是有办法的 后面会讲到用代理IP。

## 四、抓取图片

　　使用HttpClient抓取图片，先通过 entity.getContent() 获取输入流，然后 使用 common io 中的文件复制 方法 将图片专区到本地，代码如下：

### 4.1、加入依赖

```
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.5</version>
    </dependency>
```

 

### 4.2、核心代码



```
package com.jxlg.study.httpclient;

import org.apache.commons.io.FileUtils;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;

public class GetPictureByUrl {
    public static void main(String[] args) throws IOException {
        //图片路径
        String url = "https://wx2.sinaimg.cn/mw690/006RYJvjly1fmfk7c049vj30zk0qogq6.jpg";
        //创建httpClient实例
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //创建httpGet实例
        HttpGet httpGet = new HttpGet(url);
        //设置请求头消息
        httpGet.setHeader("user-Agent","Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36");
        CloseableHttpResponse response = httpClient.execute(httpGet);
        //获取.后缀
        String fileName = url.substring(url.lastIndexOf("."), url.length());

        if (response != null){
            HttpEntity entity = response.getEntity();
            if (entity != null){
                System.out.println("Content-Type:"+entity.getContentType().getValue());
                InputStream inputStream = entity.getContent();
                //文件复制
                FileUtils.copyToFile(inputStream,new File("D:love"+fileName));
            }
        }
        if (response != null){
            response.close();
        }
        if (httpClient != null){
            httpClient.close();
        }
    }
}
```

