[TOC]



### 前言

SpringBoot实现邮件功能是非常的方便快捷的，因为SpringBoot默认有starter实现了Mail。
发送邮件应该是网站的必备功能之一，什么注册验证，忘记密码或者是给用户发送营销信息。
最早期的时候我们会使用JavaMail相关api来写发送邮件的相关代码，后来spring退出了
JavaMailSender更加简化了邮件发送的过程，在之后springboot对此进行了封装就有了
现在的spring-boot-starter-mail。

## 一、基础配置

### 1.1 引入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

### 1.2 获取QQ邮箱授权码

QQ邮箱->设置->账户->POP3/SMTP服务:开启服务后会获得QQ的授权码.

### 1.3 Mail配置文件

1）qq邮箱的配置

```
# JavaMailSender 邮件发送的配置
spring.mail.host=smtp.qq.com
spring.mail.username=用户qq邮箱
#QQ邮箱的授权码
spring.mail.password=授权码
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.default-encoding=UTF-8
```

2）163邮箱配置

```
spring.mail.host=smtp.163.com
spring.mail.username=用户163邮箱
spring.mail.password=邮箱密码
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.default-encoding=UTF-8
```

## 二、实现过程

### 2.1 配置文件

```
spring:
  #邮箱配置
  mail:
    host: smtp.qq.com
    username: 917484312@qq.com
    #QQ邮箱的授权码
    password: 自己申请
    default-encoding: UTF-8
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
            required: true
  freemarker:
    content-type: text/html
    #后缀名
    suffix: .ftl
    template-loader-path: classpath:/templates
    charset: UTF-8
    check-template-location: true

  #freemarker配置




#设置一个邮件发送人
lance:
  mail:
    sender: 917484312@qq.com
```

### 2.2 实体Bean

```
/**
 * @author lance(ZYH)
 * @function  发送邮件-封装接受者信息
 * @date 2018-07-07 13:11
 */
@Data
public class MailBean implements Serializable {
    private String recipient;   //邮件接收人
    private String subject; //邮件主题
    private String content; //邮件内容

}
```

### 2.3 发送一个简单格式的邮件

```
    public void sendSimpleMail(MailBean mailBean) {
        try {
            SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
            //邮件发送人
            simpleMailMessage.setFrom(MAIL_SENDER);
            //邮件接收人
            simpleMailMessage.setTo(mailBean.getRecipient());
            //邮件主题
            simpleMailMessage.setSubject(mailBean.getSubject());
            //邮件内容
            simpleMailMessage.setText(mailBean.getContent());
            javaMailSender.send(simpleMailMessage);
        } catch (Exception e) {
            logger.error("邮件发送失败", e.getMessage());
        }
    }
```

### 2.4 发送一个HTML格式的邮件

```
    public void sendHTMLMail(MailBean mailBean) {
        MimeMessage mimeMailMessage = null;
        try {
            mimeMailMessage = javaMailSender.createMimeMessage();
            MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMailMessage, true);
            mimeMessageHelper.setFrom(MAIL_SENDER);
            mimeMessageHelper.setTo(mailBean.getRecipient());
            mimeMessageHelper.setSubject(mailBean.getSubject());
            StringBuilder sb = new StringBuilder();
            sb.append("<h1>SpirngBoot测试邮件HTML</h1>")
                    .append("\"<p style='color:#F00'>你是真的太棒了！</p>")
                    .append("<p style='text-align:right'>右对齐</p>");
            mimeMessageHelper.setText(sb.toString(), true);
            javaMailSender.send(mimeMailMessage);
        } catch (Exception e) {
            logger.error("邮件发送失败", e.getMessage());
        }
    }
```

### 2.5 发送带附件格式的邮件

```
    public void sendAttachmentMail(MailBean mailBean) {
        MimeMessage mimeMailMessage = null;
        try {
            mimeMailMessage = javaMailSender.createMimeMessage();
            MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMailMessage, true);
            mimeMessageHelper.setFrom(MAIL_SENDER);
            mimeMessageHelper.setTo(mailBean.getRecipient());
            mimeMessageHelper.setSubject(mailBean.getSubject());
            mimeMessageHelper.setText(mailBean.getContent());
            //文件路径
            FileSystemResource file = new FileSystemResource(new File("src/main/resources/static/image/mail.png"));
            mimeMessageHelper.addAttachment("mail.png", file);

            javaMailSender.send(mimeMailMessage);
        } catch (Exception e) {
            logger.error("邮件发送失败", e.getMessage());
        }
    }
```

### 2.6 发送带静态资源的邮件

```
    public void sendInlineMail(MailBean mailBean) {
        MimeMessage mimeMailMessage = null;
        try {
            mimeMailMessage = javaMailSender.createMimeMessage();
            MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMailMessage, true);
            mimeMessageHelper.setFrom(MAIL_SENDER);
            mimeMessageHelper.setTo(mailBean.getRecipient());
            mimeMessageHelper.setSubject(mailBean.getSubject());
            mimeMessageHelper.setText("<html><body>带静态资源的邮件内容，这个一张IDEA配置的照片:<img src='cid:picture' /></body></html>", true);
            //文件路径
            FileSystemResource file = new FileSystemResource(new File("src/main/resources/static/image/mail.png"));
            mimeMessageHelper.addInline("picture", file);

            javaMailSender.send(mimeMailMessage);
        } catch (Exception e) {
            logger.error("邮件发送失败", e.getMessage());
        }
    }
```

### 2.7 发送基于Freemarker模板的邮件

```
    public void sendTemplateMail(MailBean mailBean) {
        MimeMessage mimeMailMessage = null;
        try {
            mimeMailMessage = javaMailSender.createMimeMessage();
            MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMailMessage, true);
            mimeMessageHelper.setFrom(MAIL_SENDER);
            mimeMessageHelper.setTo(mailBean.getRecipient());
            mimeMessageHelper.setSubject(mailBean.getSubject());

            Map<String, Object> model = new HashMap<String, Object>();
            model.put("content", mailBean.getContent());
            model.put("title", "标题Mail中使用了FreeMarker");
            Template template = configuration.getTemplate("mail.ftl");
            String text = FreeMarkerTemplateUtils.processTemplateIntoString(template, model);
            mimeMessageHelper.setText(text, true);

            javaMailSender.send(mimeMailMessage);
        } catch (Exception e) {
            logger.error("邮件发送失败", e.getMessage());
        }

    }
```

效果：

### 2.8 发送基于 thymeleaf为模板的邮件

1）pom中导入thymeleaf的包

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2）在resorces/templates下创建emailTemplate.html

```
<!DOCTYPE html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8"/>
        <title>Title</title>
    </head>
    <body>
        您好,这是验证邮件,请点击下面的链接完成验证,<br/>
        <a href="#" th:href="@{ http://www.ityouknow.com/neo/{id}(id=${id}) }">激活账号</a>
    </body>
</html>
```

3）解析模板并发送

```
@Test
public void sendTemplateMail() {
    //创建邮件正文
    Context context = new Context();
    context.setVariable("id", "006");
    String emailContent = templateEngine.process("emailTemplate", context);

    mailService.sendHtmlMail("ityouknow@126.com","主题：这是模板邮件",emailContent);
}
```

## 三、邮件服务的问题

### 3.1 邮件发送失败

因为各种原因，总会有邮件发送失败的情况，比如：邮件发送过于频繁、网络异常等。在出现这种情况的时候，
我们一般会考虑重新重试发送邮件，会分为以下几个步骤来实现：

- 接收到发送邮件请求，首先记录请求并且入库。
- 调用邮件发送接口发送邮件，并且将发送结果记录入库。
- 启动定时系统扫描时间段内，未发送成功并且重试次数小于3次的邮件，进行再次发送。

### 3.2 邮件异步发送

很多时候邮件发送并不是我们主业务必须关注的结果，比如通知类、提醒类的业务可以允许延时或者失败。
这个时候可以采用异步的方式来发送邮件，加快主交易执行速度，在实际项目中可以采用MQ发送邮件相关
参数，监听到消息队列之后启动发送邮件。

