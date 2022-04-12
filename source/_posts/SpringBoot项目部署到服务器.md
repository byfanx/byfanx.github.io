---
title: SpringBoot项目部署到服务器
route: springboot-deploy-server
date: 2019-12-23 14:20:01
tags: [SpringBoot,服务器]
categories: SpringBoot
image: /images/cover/springboot-deploy-server.png
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在之前的博客当中，写了一些关于SpringBoot的使用，今天主要写的就是怎么样将自己写的SpringBoot项目部署到云服务器上。只有将项目部署到了服务器上，项目才能一直运行，并且只要在有互联网就能随时随地的的访问项目。

# SpringBoot项目部署到服务器

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在部署之前呢，先要再简单了解一下SpringBoot项目的运行。因为SpringBoot框架默认自带了一个嵌入式的Tomcat服务器，所以可以通过自带的服务器以jar包的方法运行，也可以将SpringBoot项目打包成一个war包，部署到服务器的Tomcat上运行。以下是对两种运行方式部署到服务器上进行详解，需要注意的时，这个项目时一个maven项目。

## 1 使用War包部署

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 使用war包部署，就只需要将项目打包，部署到服务器Tomcat下即可运行，不过在打包之前，需要对项目做一些简单的修改，步骤如下。

### 1.1 修改pom.xml文件

打开项目的pom.xml配置文件，SpringBoot默认的打包方式是jar包，将打包方式设置为war。

```xml
<packaging>war</packaging>
```

### 1.2 修改启动类

对于启动类，需要重写初始化方法：继承SpringBootServletInitializer，重写configure函数。

```java
@SpringBootApplication
public class FilmManageApplication extends SpringBootServletInitializer{
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return super.configure(builder);
    }

    public static void main(String[] args) {
        SpringApplication.run(FilmManageApplication.class, args);
    }
}  
```

### 1.3 去除自带的Tomcat

方法：需要将嵌入的Tomcat依赖方式改成provide(编译、测试时将依赖的包加入本工程的classpath，运行时不加入。可以理解成运行时不适使用SpringBoot自带的Tomcat)。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

### 1.4 加入servlet-api依赖

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```

### 1.5 打包

点击IDEA右侧的Maven，双击package打包，注意打包时跳过测试，再target目录下生成war包文件。

![打包war](打包war.png)

### 1.6 部署

将打包好的war包文件放到云服务器**Tomcat的webapps目录**下，启动Tomcat(在bin目录下执行./startup.sh)，即可自动解压部署。

### 1.7 测试

项目部署好之后，打开浏览器输入服务器地址:端口号/项目jar包名/主页。

![测试](测试war.png)

### 1.8 说明

1. 由于war包部署不使用SpringBoot自带的Tomcat，所以application.yml文件下的server配置将不会再起作用，故服务器默认Tomcat端口号时**8080**。想要修改端口号可以到Tomcat下的**server.xml**文件进行修改。路径输入的是项目war包的名称。

2. 默认情况下，项目路径是war包或jar包的名称，如果想要修改的话，可以再Tomcat里的**server.xml**文件中加上

   ```xml
   <Context docBase="MyProject.war" path="/demo" reloadable="true" debug="0" privileged="true">
   ```

   输入地址时就可以直接用demo代替MyProject。

3. 注意：如果在server.xml中加了以上语句的话，那么需要特别注意的就是，该项目的war包不能被删除，否则Tomcat将无法启动。

## 2 使用Jar包部署 

### 2.1 打包

​        首先，如果没有在pom.xml文件中修改默认的package的话，默认的就是以jar方式打包。然后点击IDEA右侧的Maven，跳过测试，package打包。

![打包jar](打包jar.png)

### 2.2 部署

​        之后在target目录下找到打包之后的jar文件，然后通过工具上传至云服务器。

### 2.3 运行

​        如果服务器是windows server桌面版的，将jar包上传之后，可以直接双击jar包运行，不过关闭时需要到任务管理器关闭改任务。

​         如果是Linux版本的服务器，那么上传之后，在jar包所在的目录下，输入命令：

```shell
java -jar XXX.jar
```

​        如果想指定运行端口也可以，输入以下命令。不过个人觉得没有必要，因为已经在配置文件里指定项目端口了。

```shell
java -jar XXX.jar --server.port=8088
```

​        输入命令回车，就会出现如图所示，程序已经启动。

![运行jar](运行jar.png)

### 2.4 测试

​        之后在浏览器中输入服务器地址:在application.yml中设置的端口号/jar包名称/主页。

![测试jar](测试jar.png)

### 2.5 说明

​        利用上面所说的方式，关闭远程端口程序就会自动退出，如果需要后台运行，可以使用下列命令进行部署。

**<1>首次部署**

```shell
nohup java -jar XXX.jar >temp.text &
//退出 ctrl+c
```

其中：

+ nohup：当账户退出或者终端关闭时，程序仍然运行。
+ &：客户端关闭，后台停止运行。
+ temp.text 是存控制台文件。缺省情况下该作业的所有的输出内容被重定向到nohup.out的文件中。
+ 使用tail -f temp.text实时查看控制台文件。ctrl+z返回命令行。

可通过`jobs`命令列出所有后台运行任务，并且每个作业前面都有编号，如果想将某个作业调回前台控制，只需要`fg + 编号`即可。

**<2>非首次部署**

非首次部署当前程序需要在对应的文件夹中执行以下命令：

a.捕获上一个版本程序的进程 

```shell
ps -ef|grep XXX.jar
```

b.杀死对应的进程

```shell
kill 进程号
```

c.启动程序

```shell
nohup java -jar XXX.jar >temp.text &
```

d.退出

```shell
ctrl + c
```

e.查看日志

```shell
tail -500f temp.text
```

## 3.总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上就是将SpringBoot项目部署到云服务器的全部过程，简单总结就是jar包部署方式使用SpringBoot自带的Tomcta，所以可以直接在服务器运行jar文件。war包部署方式则使用云服务器里的Tomcat，此时需要将SpringBoot自带的Tomcat移除，并在打包时对项目进行一点小的修改。同时要注意的就是两种部署方式的访问路径的差异，就是端口的不同，一个使用application.yml里配置的端口，一个使用Tomcat默认的端口号。