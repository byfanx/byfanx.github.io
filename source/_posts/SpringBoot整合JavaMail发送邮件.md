---
title: SpringBoot整合JavaMail发送邮件
route: SpringBoot-JavaMail
tags: [SpringBoot,Java]
date: 2022-01-27 17:04:50
categories: SpringBoot
image: /images/cover/SpringBoot-JavaMail.jpeg
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在SpringBoot项目的开发中，遇到过发送邮件的功能，如果用原始写法的话需要配置大量的东西，现在可以可以直接用SpringBoot整合的JavaMail来做这件事。
<!-- more -->

#  SpringBoot整合JavaMail发送邮件

##  1. 前提了解

在整合之前呢，需要先简单了解一下相关协议，这里有三个：

- SMTP（Simple Mail Transfer Protocol）：简单邮件传输协议，用于**发送**电子邮件的传输协议。

- POP3（Post Office Protocol 3）：用于**接收**电子邮件的标准协议。

- IMAP（Internet Mail Access Protocol）：互联网消息协议，是POP3的替代协议。


对于我们开发而言，一般用的最多的就是发送邮件。

## 2. 导入相关坐标

因为SpringBoot已经整合了JavaMail，所以这里版本号可以省略不写，使用SpringBoot默认的就行。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

## 3. 配置文件

在`application.properties`或者`application.yml`中配置相关信息，这里用的是`application.yml`文件格式。

```yaml
spring:
  mail:
    host: smtp.163.com			# 邮箱服务器
    username: test@163.com		# 用户名					
    password: s4ty34fg75nu985	# 邮箱的授权码，而不是邮箱密码
```

> 邮箱的授权码获取方式见文章末尾的[附录](#fulu)。

## 4. 编写代码

### 4.1 发送简单邮件

编写测试方法，首先通过注解注入`JavaMailSender`对象，然后就可以通过该类进行邮件发送的操作。代码如下。

```java
// 注入实体类
@Autowired
private JavaMailSender mailSender;

/**
 * 发送简单邮件
 * @param from 发送者  xxx@163.com
 * @param to 接收者    xxx@163.com
 * @param subject 邮件标题
 * @param context 邮件正文
 */
public void sendSimpleMail(String from, String to, String subject, String context){
    SimpleMailMessage message = new SimpleMailMessage();
    // 设置发送者
    message.setFrom(from);
    // 下面这种发送方式，就是在邮箱后面通过括号加一个昵称，在邮件列表中可以显示收件人为昵称，而不是直接显示邮箱
    // message.setFrom(from + "(byfan)");

    // 设置接收者
    message.setTo(to);
    // 设置标题
    message.setSubject(subject);
    // 设置正文
    message.setText(context);
    // 设置发送时间，可用于定时发送
    // message.setSentDate(new Date());

    // 发送邮件
    mailSender.send(message);

}
```

### 4.2 发送多部件邮件

```java
// 注入实体类
@Autowired
private JavaMailSender mailSender;

/**
 * 发送简单邮件
 * @param from 发送者  xxx@163.com
 * @param to 接收者    xxx@163.com
 * @param subject 邮件标题
 * @param context 邮件正文 eg:<a href='https://byfan.xyz'>点击进入byFan驿站</a>
 * @param isHtml 是否解析正文为html
 * @param files 附件列表
 */
public void sendMimeMail(String from, String to, String subject, String context, boolean isHtml, File[] files){
    try {
        // 定义是否有附件的标识
        boolean isFile = false;
        MimeMessage message = mailSender.createMimeMessage();
        if (files.length > 0){
            isFile = true;
        }
        // 第二个参数可不传，默认是false不带附件
        MimeMessageHelper helper = new MimeMessageHelper(message,isFile);
        // 设置发送者
        helper.setFrom(from);
        // 下面这种发送方式，就是在邮箱后面通过括号加一个昵称，在邮件列表中可以显示收件人为昵称，而不是直接显示邮箱
        // message.setFrom(from + "(byfan)");

        // 设置接收者
        helper.setTo(to);
        // 设置标题
        helper.setSubject(subject);
        // 设置正文,第二个参数可不传，默认是false不解析html
        helper.setText(context,isHtml);

        // 添加附件
        if (isFile){
            for (File file : files){
                // 第一个参数：附件文件名，可自定义；第二个参数：附件文件
                helper.addAttachment(file.getName(), file);
            }
        }

        // 设置发送时间，可用于定时发送
        // message.setSentDate(new Date());

        // 发送邮件
        mailSender.send(message);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

<span id="fulu"></span>

## 附录：获取邮箱授权码

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;获取邮箱授权码这里也以163邮箱为例，其他邮箱获取授权码的步骤也都差不都，可做参考。首先登录到163邮箱网页版。在上面导航栏中找到设置，点击**设置**，然后点击**POP3/SMTP/IMAP**。进入到操作页面之后，在**开启服务**项里面，点击**IMAP/SMTP**后面的**开启**，弹窗提示需要发送短信进行验证，按照提示发送完成之后，点击**我已发送**，则会弹窗显示当前授权码，**该授权码只显示一次，记得保存备份。**

![](image-20220127173751861.png)



![](image-20220127174246154.png)