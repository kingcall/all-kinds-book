前言

　　其实前面写的那一点点东西都是轻轻点水，其实HttpClient还有很多强大的功能：

　　（1）实现了所有 HTTP 的方法（GET,POST,PUT,HEAD 等）

　　（2）支持自动转向

　　（3）支持 HTTPS 协议

　　（4）支持代理[服务器](https://baike.baidu.com/item/服务器)等

## 一、HttpClient使用代理IP

### 1.1、前言

　　在爬取网页的时候，有的目标站点有反爬虫机制，对于频繁访问站点以及规则性访问站点的行为，会采集屏蔽IP措施。
　　这时候，代理IP就派上用场了。可以使用代理IP，屏蔽一个就换一个IP。
　　关于代理IP的话 也分几种 透明代理、匿名代理、混淆代理、高匿代理，一般使用高匿代理。　　　　

### 1.2、几种代理IP

　　1）透明代理（Transparent Proxy）

　　　　REMOTE_ADDR = Proxy IP
　　　　HTTP_VIA = Proxy IP
　　　　HTTP_X_FORWARDED_FOR = Your IP
　　　　透明代理虽然可以直接“隐藏”你的IP地址，但是还是可以从HTTP_X_FORWARDED_FOR来查到你是谁。

　　2）匿名代理(Anonymous Proxy)

　　　　REMOTE_ADDR = proxy IP
　　　　HTTP_VIA = proxy IP
　　　　HTTP_X_FORWARDED_FOR = proxy IP
　　　　匿名代理比透明代理进步了一点：别人只能知道你用了代理，无法知道你是谁。
　　　　还有一种比纯匿名代理更先进一点的：混淆代理

　　3）混淆代理(Distorting Proxies)

　　　　REMOTE_ADDR = Proxy IP
　　　　HTTP_VIA = Proxy IP
　　　　HTTP_X_FORWARDED_FOR = Random IP address
　　　　如上，与匿名代理相同，如果使用了混淆代理，别人还是能知道你在用代理，但是会得到一个假的IP地址，伪装的更逼真。

　　4）高匿代理(Elite proxy或High Anonymity Proxy)

　　　　REMOTE_ADDR = Proxy IP
　　　　HTTP_VIA = not determined
　　　　HTTP_X_FORWARDED_FOR = not determined
　　　　可以看出来，高匿代理让别人根本无法发现你是在用代理，所以是最好的选择。
　　　　一般我们搞爬虫 用的都是 高匿的代理IP；
　　　　那代理IP 从哪里搞呢 很简单 百度一下，你就知道 一大堆代理IP站点。 一般都会给出一些免费的，但是花点钱搞收费接口更加方便。

### 1.3、实例来使用代理Ip

　　使用 RequestConfig.custom().setProxy(proxy).build() 来设置代理IP　　



```
package com.jxlg.study.httpclient;

import com.sun.org.apache.regexp.internal.RE;
import org.apache.http.HttpEntity;
import org.apache.http.HttpHost;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.IOException;

public class UseProxy {
    public static void main(String[] args) throws IOException {
        //创建httpClient实例
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //创建httpGet实例
        HttpGet httpGet = new HttpGet("http://www.tuicool.com");
        //设置代理IP，设置连接超时时间 、 设置 请求读取数据的超时时间 、 设置从connect Manager获取Connection超时时间、
        HttpHost proxy = new HttpHost("58.60.255.82",8118);
        RequestConfig requestConfig = RequestConfig.custom()
                .setProxy(proxy)
                .setConnectTimeout(10000)
                .setSocketTimeout(10000)
                .setConnectionRequestTimeout(3000)
                .build();
        httpGet.setConfig(requestConfig);
        //设置请求头消息
        httpGet.setHeader("User-Agent","Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36");
        CloseableHttpResponse response = httpClient.execute(httpGet);

        if (response != null){
            HttpEntity entity = response.getEntity();  //获取返回实体
            if (entity != null){
                System.out.println("网页内容为:"+ EntityUtils.toString(entity,"utf-8"));
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



### 1.4、实际开发中怎么去获取代理ip

　　我们可以使用HttpClient来 爬取 http://www.xicidaili.com/ 上最新的20条的高匿代理IP，来保存到 链表中，当一个IP被屏蔽之后获取连接超时时，

　　就接着取出 链表中的一个IP，以此类推，可以判断当链表中的数量小于5的时候，就重新爬取 代理IP 来保存到链表中。

### 1.5、HttpClient连接超时及读取超时

　　httpClient在执行具体http请求时候 有一个连接的时间和读取内容的时间；

　　1)HttpClient连接时间

　　　　所谓连接的时候 是HttpClient发送请求的地方开始到连接上目标url主机地址的时间，理论上是距离越短越快，

　　　　线路越通畅越快，但是由于路由复杂交错，往往连接上的时间都不固定，运气不好连不上，HttpClient的默认连接时间，据我测试，

　　　　默认是1分钟，假如超过1分钟 过一会继续尝试连接，这样会有一个问题 假如遇到一个url老是连不上，会影响其他线程的线程进去，说难听点，

　　　　就是蹲着茅坑不拉屎。所以我们有必要进行特殊设置，比如设置10秒钟 假如10秒钟没有连接上 我们就报错，这样我们就可以进行业务上的处理，

　　　　比如我们业务上控制 过会再连接试试看。并且这个特殊url写到log4j日志里去。方便管理员查看。

　　2）HttpClient读取时间

　　　　所谓读取的时间 是HttpClient已经连接到了目标服务器，然后进行内容数据的获取，一般情况 读取数据都是很快速的，

　　　　但是假如读取的数据量大，或者是目标服务器本身的问题（比如读取数据库速度慢，并发量大等等..）也会影响读取时间。

　　　　同上，我们还是需要来特殊设置下，比如设置10秒钟 假如10秒钟还没读取完，就报错，同上，我们可以业务上处理。　　　　　

　　比如我们这里给个地址 http://central.maven.org/maven2/，这个是国外地址 连接时间比较长的，而且读取的内容多 。很容易出现连接超时和读取超时。　　

　　我们如何用代码实现呢？

　　HttpClient给我们提供了一个RequestConfig类 专门用于配置参数比如连接时间，读取时间以及前面讲解的代理IP等。

　　例子：



```
package com.jxlg.study.httpclient;

import org.apache.http.HttpEntity;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.IOException;

public class TimeSetting {
    public static void main(String[] args) throws IOException {
        CloseableHttpClient httpClient = HttpClients.createDefault();
        HttpGet httpGet = new HttpGet("http://central.maven.org/maven2/");
        RequestConfig config = RequestConfig.custom()
                .setConnectTimeout(5000)
                .setSocketTimeout(5000)
                .build();
        httpGet.setConfig(config);
        httpGet.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36");
        CloseableHttpResponse response = httpClient.execute(httpGet);
        if (response != null){
            HttpEntity entity = response.getEntity();
            System.out.println("网页内容为:"+ EntityUtils.toString(entity,"UTF-8"));
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

 

　　

 