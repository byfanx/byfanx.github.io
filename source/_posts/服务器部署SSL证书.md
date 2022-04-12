---
title: 服务器部署SSL证书
route: server-SSL
date: 2019-10-6 18:00:32
tags: [服务器,其他]
categories: 服务器
image: /images/cover/server-SSL.png
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有时候项目需要服务器的协议是`https`协议，比如微信小程序的后台接口；这个时候就不得不把服务器协议修改为`https`。`https`和`http`有什么区别呢？区别就在于`https`比`http`多一个SSL证书，下面就是怎么获取SSL证书和怎么部署到服务器上。(唉，这也是当年自己踩了两个小时坑踩出来的啊！)
<!-- more -->

# 服务器部署SSL证书

## 1. 获取SSL证书
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为当初我是服务器是在阿里云上租的，是**Window Server**所以SSL证书也是在阿里云上面获取的，当然，不要想着我会用钱去买这玩意，主要是太贵了，不过阿里云也算良心，能在上面免费获取一个一年的SSL证书。在这免费获取[阿里云SSL证书](https://www.aliyun.com/product/cas?spm=5176.12825654.eofdhaal5.19.3dbd2c4ayD5k2Q&aly_as=nsmL9Pxx)，打开连接之后选择**证书对比**，里面有一个免费证书可以立即购买，阿里云免费证书是*Symantec*颁发的。购买之后在SSL证书控制台进行查看自己的证书，然后提交审核，有时很几分钟就审核完了，有时候要好几天，这就要看运气了。审核通过之后在SSL证书列表最后面点击下载，然后选择自己对应的服务器类型。下载下来的是一个压缩包。

## 2. 部署到Tomcat

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下载的压缩包里面有两个文件，一个是`.pfx`文件，这个就是我们的SSL证书；一个是`.txt`文件，这个是证书所对应的密码。此时SSL证书就到手了，接下来就可以进行部署了。将我们下载的证书部署到Tomcat上，首先需要通过Java jdk将**pfx**证书转换为**jks**证书，先将我们的**prx**证书放到jdk的bin目录下。然后通过cmd打开命令行窗口，进入到jdk的bin目录下，假设我们的证书是**a.pfx**，输入以下命令：
```cmd
keytool -importkeystore -srckeystore a.pfx -destkeystore a.jks -srcstoretype PKCS12 -deststoretype JKS
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;回车后输入JKS证书密码和PFX证书密码，强烈推荐将JKS密码与PFX证书密码相同，否则可能会导致Tomcat启动失败。
运行截图如下：
![](jks.png)

>**注意**: pfx文件必须放到你的jdk的 bin目录下面哦，并且它生成的jks文件也会在此目录下面的哦。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有了jks文件之后，下面就是对Tomcat的操作了，首先将刚刚获得的**jks**文件放到Tomcat的**conf**目录下，然后打开Tomcat的**server.xml**文件，也是在conf目录下，需要修改以下三个地方。
1.把

```xml
<Connector port="8080" protocol="HTTP/1.1" address="0.0.0.0"
    connectionTimeout="20000"
    redirectPort="8443"  />
```
改成
```xml
<Connector port="80" protocol="HTTP/1.1" address="0.0.0.0"
    connectionTimeout="20000"
    redirectPort="443"  />
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其中第一个80端口是为HTTP(HyperText Transport Protocol)即超文本传输协议开放的，此为上网冲浪使用次数最多的协议，第二个443端口是SSL的专用端口，HTTPS监听的端口是443端口。
2.把

```xml
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```
改成
```xml
<Connector port="8009" protocol="AJP/1.3" redirectPort="443" />
```
使用443端口的理由同上。
3.把

```xml
<!--
<Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true">
        <SSLHostConfig>
            <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"
                    type="RSA" />
        </SSLHostConfig>
    </Connector>
-->
```
改成
```xml
<Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true">
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="conf/a.jks"
                     certificateKeystorePassword="IBdiYc8W"
                     type="RSA" />
    </SSLHostConfig>
</Connector>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先去掉注释，然后certificateKeystoreFile属性是让你告诉服务器需要哪个SSL证书，后面就填复制过去的那个jks文件的名字（记得带上jks后缀），然后加上certificateKeystorePassword这个属性，后面的属性值填秘钥。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;到这，关于server.xml文件的配置就完成了。

## 3. 部署到Nginx

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在阿里云下载的压缩包里面有两个文件，一个是`.pem`文件，一个是`.key`文件。[腾讯云的免费证书](https://console.cloud.tencent.com/ssl/apply)是*TrustAsia*颁发的，下载的压缩包里面有两个文件，一个是`.crt`文件，一个是`.key`文件。二者只是证书颁发商不同，实际都是可以满足需求的。这里选用的是阿里云申请的证书，进入到nginx的安装目录下，在`conf`目录下创建`cert`目录，然后将解压之后的文件放到该目录下。

![](nginxSSL.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开`nginx.conf`配置文件，添加以下配置：

```json
server {

        listen 443 ssl;  #新版本启用SSL功能。
    	# ssl on;   旧版本设置为on开启ssl功能
        server_name example.com;  # 替换成你的域名
        ssl_certificate cert/2837922_example.com.pem;   #替换成你的pem/crt文件名称
        ssl_certificate_key cert/2837922_example.com.key;   #替换成你的key文件名称
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  #使用此加密套件。
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   #使用该协议进行配置。
        ssl_prefer_server_ciphers on;  


        # listen       80;
        # server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
}
```

然后重启nginx服务器即可通过https访问服务器。

也可以使用全站加密，对于用户不知道网站可以进行 https 访问的情况下，让服务器自动把 http 的请求重定向到 https。在`nginx.conf`文件中添加以下配置：

```json
server {
        listen       80;
        server_name  example.com;#替换成你的域名


        location / {
            # 配置请求转发，将http的请求转发到https上面
            rewrite ^(.*)$ https://example.com/$1 permanent;#替换成你的域名
        }
    }
```



## 4. 附加：开启SSL443端口

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有时候我们完成以上工作之后，启动服务器依然没有办法访问到，这可能是因为云服务器的SSL端口没有打开。
1.打开"控制面板"-->"系统和安全"-->"Windows防火墙"

2.选择打开活关闭防火墙
![开启防火墙](1.png "开启防火墙")

3.启用Windows防火墙
![启用防火墙](2.png "启用防火墙")

4.选择"高级设置"
![高级设置](3.png "高级设置")

5.依次点击"入站规则"-->"新建规则"

6.选择"端口"，然后"下一步"
![新建规则](4.png "新建规则")

7.选择"特定本地端口"，然后再后面的框中填入443(当然选择所有本地端口也可以开启443端口，但是端口全部开放可能会造成一定隐患，不建议)，然后"下一步"
![](5.png)

8.选择"允许连接"，继续"下一步"
![](6.png)

9.默认三个全部勾选，下一步
![](7.png)

10.填写规则名称，建议填一个容易辨识的名字，描述可填可不填，点击完成
![](8.png)

11.此时端口启用成功
![](9.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;至此，所有配置和部署就已经全部完成了，接下来就可以通过**https**访问网站了。
![https访问](10.png)