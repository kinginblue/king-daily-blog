# Spring Boot 发送邮件

## 一|网上已有的文章已经讲得差不多了

[Sending Email](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-email.html)

[Guide to Spring Email](http://www.baeldung.com/spring-email)

[Spring Boot中使用JavaMailSender发送邮件](http://blog.didispace.com/springbootmailsender/)

## 二|SpringBoot 下总体使用过程

pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
    <version>1.4.3.RELEASE</version>
</dependency>
```

application.yml

```yaml
spring:
  mail:
    protocol: smtp
    host: <>
    port: 25
    username: <>
    password: <>
    properties:
      mail:
        smtp:
          auth: true
          connectiontimeout: 5000
          timeout: 5000
          writetimeout: 5000
          starttls:
            enable: true
            required: true
```

`MailSenderAutoConfiguration` 在SpringBoot 的自动配置机制，会自动配置一个 `JavaMailSender` Bean， 直接注入就可以使用了：

```java
@Autowired
private JavaMailSender mailSender;
.....
```

发送邮件时，邮件 `发送者` 要与前去邮件服务器认证的账户一致，即上面的 `spring.mail.username`，代码中可以这样获取：

```java
@Autowired
private MailProperties mailProperties;

String from = mailProperties.getUsername();
```

发送简单邮件的示例代码：

```java
SimpleMailMessage message = new SimpleMailMessage();
message.setFrom(mailProperties.getUsername());
message.setTo("qq@qq.com"));
message.setSubject("mail subject");
message.setText("mail text");
mailSender.send(message);
```

## 三|遇到的问题

在使用腾讯企业邮箱发送邮件时，总是提示：

```java
javax.mail.MessagingException: Exception reading response;
  nested exception is:
        java.net.SocketTimeoutException: Read timed out
```

使用的配置如下：

```yaml
spring:
  mail:
    protocol: smtp
    host: smtp.exmail.qq.com
    port: 465
    username: admin@gg.io
    password: thepassword
    properties:
      mail:
        smtp:
          auth: true
          connectiontimeout: 5000
          timeout: 5000
          writetimeout: 5000
          starttls:
            enable: true
            required: true
```

搜到 stackoverflow 上这个文章 [Error when sending email via Java Mail API?](https://stackoverflow.com/questions/31535863/error-when-sending-email-via-java-mail-api)

把 `port: 465` 改成 `port: 587` 就可以了！

解释是：

     Port 465 expects you to start an SSL session and then run smtp over it, compared to port 587 where the session begins in plaintext SMTP, and then starttls is used to negotiate an encrypted TLS session. So, the (port: 465 and starttls: enable: true) of your config  are in conflict.