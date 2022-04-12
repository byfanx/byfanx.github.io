---
title: Spring定时任务-@Scheduled注解
route: spring-Scheduled
date: 2019-9-2 22:13:32
tags: [Spring,Java]
categories: Spring
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有时候项目中需要用到定时任务，在某个时间点执行特定的任务，这时我们就可以使用spring中的定时任务——**@Scheduled注解**。下面就简单介绍一下`@Scheduledd`注解的使用。在springMVC项目中使用spring的定时任务步骤如下。
<!-- more -->
# @Scheduled注解详解
## 1. 配置springMVC.xml文件
加入tesk的命名空间:
```xmls
xmlns:task = "http://www.springframework.org/schema/task"
```

在xsi:schemaLocation中添加以下依赖:
```java
http://www.springframework.org/schema/task
http://www.springframework.org/schema/task/spring-task-4.0.xsd"
```

## 2. 配置定时任务的线程池
推荐配置线程池，若不配置，多任务下会有问题。
```xml
<task:scheduler id="myScheduler" pool-size="5"/>
```
>id是该线程池的唯一标识，用于启动该线程池的注解驱动。poo-size是线程池的容量。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spring的定时任务默认是单线程，多个任务执行起来时间会有问题。在下面定时任务的示例代码中，A任务中设置的有`TimeUnit.SECONDS.sleep(20);`那么如存在B任务，B任务会因为A任务执行起来需要20s二被延后20s执行。当我们配置了线程池之后，多线程下B任务就不会因为A任务执行起来要20s而别延后执行了。

## 3. 启动注解驱动的定时任务
```xml
<task:annotation-driven scheduler="myScheduler"/> 
```
> scheduler指的是我们配置线程池的id名称，如果没有配置线程池，那么该元素可以不写。

## 4. 配置扫描包
```xml
<context:component-scan base-package="test"/>-->
```
> base-package元素是配置定时任务所在的包名。

## 5. 编写定时任务
`@Scheduled`注解为定时任务，`cron`表达式里写执行的时机。以下代码是示例代码。
```java
package test;
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.concurrent.TimeUnit;
import org.joda.time.DateTime;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
@Component
public class ATask{
       @Scheduled(cron="0/10 * *  * * ? ")   //每10秒执行一次  
       public void aTask(){ 
            try {
                    TimeUnit.SECONDS.sleep(20);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            DateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
            System.out.println(sdf.format(DateTime.now().toDate())+"*********A任务每10秒执行一次进入测试");    
       }    
}  
```
> @Component泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。    

> 有时候启动web项目发现随着服务器启动的定时任务没有执行，这是因为xml的头文件`<beans>`里面的一个配置default-lazy-init="true"。此处配置为懒加载，所以所有的bean都是懒加载，导致定时任务所在的bean根本没有实例化，里面的定时任务也没有执行，修改为default-lazy-init="false"即可。 

## 6. @Scheduled注解各参数详解

### 6.1 cron表达式详解

该参数接收一个`cron表达式`，`cron表达式`是一个字符串，字符串以5个或6个空格隔开，分开共6个或7个域，每个域代表一个含义

#### 6.1.1 cron表达式语法
```java
[秒] [分] [时] [日] [月] [周] [年]
```
>注：[年]不是必须的域，可以省略年，则一共6个域。

序号|说明|必填|允许填写的值|允许的通配符
:-:|:-:|:-:|:-:|:-:
1|秒|是|0-59|, - * /
2|分|是|0-59|, - * /
3|时|是|0-23|, - * /
4|日|是|1-31|, - * / L W
5|月|是|1-12/JAN-DEC|, - * /
6|周|是|1-7/SUN-SAT|, - * / L #
7|年|否|1970-2099|, - * /

#### 6.1.2 通配符说明
+ `*` 表示所有值。 例如:在分的字段上设置 *,表示每一分钟都会触发。
+ `?` 表示不指定值。使用的场景为不需要关心当前设置这个字段的值。例如:要在每月的10号触发一个操作，但不关心是周几，所以需要周位置的那个字段设置为”?” 具体设置为 0 0 0 10 * ?
+ `-` 表示区间。例如 在小时上设置 “10-12”,表示 10,11,12点都会触发。
+ `,` 表示指定多个值，例如在周字段上设置 “MON,WED,FRI” 表示周一，周三和周五触发
+ `/` 用于递增触发。如在秒上面设置”5/15” 表示从5秒开始，每增15秒触发(5,20,35,50)。 在月字段上设置’1/3’所示每月1号开始，每隔三天触发一次。
+ `L` 表示最后的意思。在日字段设置上，表示当月的最后一天(依据当前月份，如果是二月还会依据是否是润年[leap]), 在周字段上表示星期六，相当于“7”或“SAT”。如果在“L”前加上数字，则表示该数据的最后一个。例如在周字段上设置“6L”这样的格式,则表示“本月最后一个星期五”
+ `W` 表示离指定日期的最近那个工作日(周一至周五). 例如在日字段上置“15W”，表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发.如果15号正好在工作日(周一至周五)，则就在该天触发。如果指定格式为 “1W”,它则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，“W”前只能设置具体的数字,不允许区间“-”)。
+ `#` 序号(表示每月的第几个周几)，例如在周字段上设置“6#3”表示在每月的第三个周六.注意如果指定“#5”,正好第五周没有周六，则不会触发该配置(用在母亲节和父亲节再合适不过了) ；小提示：‘L’和 ‘W’可以一组合使用。如果在日字段上设置“LW”,则表示在本月的最后一个工作日触发；周字段的设置，若使用英文字母是不区分大小写的，即MON与mon相同。

**示例:**
+ 每隔5秒执行一次：*/5 * * * * ?

+ 每隔1分钟执行一次：0 */1 * * * ?

+ 每天23点执行一次：0 0 23 * * ?

+ 每天凌晨1点执行一次：0 0 1 * * ?

+ 每月1号凌晨1点执行一次：0 0 1 1 * ?

+ 每月最后一天23点执行一次：0 0 23 L * ?

+ 每周星期天凌晨1点实行一次：0 0 1 ? * L

+ 在26分、29分、33分执行一次：0 26,29,33 * * * ?

+ 每天的0点、13点、18点、21点都执行一次：0 0 0,13,18,21 * * ?

### 6.2 zone
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;时区。接收一个`java.util.TimeZone#ID`。`cron表达式`会基于该时区解析。默认是一个空字符串，即取服务器所在地的时区。比如我们一般使用的时区`Asia/Shanghai`。该字段我们一般留空。

### 6.3 fixedDelay
上一次执行完毕时间点之后多长时间再执行。如：
```java
@Scheduled(fixedDelay = 5000) //上一次执行完毕时间点之后5秒再执行
```

### 6.4 fixedDelayString
与`fixedDelay`意思相同，只是使用字符串的形式。唯一不同的是支持占位符。如：
```java
@Scheduled(fixedDelayString = "5000") //上一次执行完毕时间点之后5秒再执行
```
占位符的使用(配置文件中有配置：time.fixedDelay=5000)：
```java
@Scheduled(fixedDelayString = "${time.fixedDelay}")
void testFixedDelayString() {
    System.out.println("Execute at " + System.currentTimeMillis());
}
```

### 6.5 fixedRate
上一次开始执行时间点之后多长时间再执行。如：
```java
@Scheduled(fixedRate = 5000) //上一次开始执行时间点之后5秒再执行
```

### 6.6 fixedRateString
与`fixedRate`意思相同，只是使用字符串的形式。唯一不同的是支持占位符。

### 6.7 initialDelay
第一次延迟多长时间后再执行。如：
```java
@Scheduled(initialDelay=1000, fixedRate=5000) //第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次
```

### 6.8 initialDelayString
与`initialDelay`意思相同，只是使用字符串的形式。唯一不同的是支持占位符。
