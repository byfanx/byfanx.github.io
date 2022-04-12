---
title: SpringBoot-整合Thymeleaf
route: springboot-thymeleaf
date: 2019-10-24 19:16:20
tags: [SpringBoot,Java,前端]
categories: SpringBoot
image: /images/cover/springboot-thymeleaf.png
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;学习完SpringBoot之后，以后写web项目就用它了，以前开发web项目使用的还是JSP页面，但是SpringBoot官方是不支持JSP的，它默认支持的模板是Thymeleaf，既然学习了SpringBoot，怎么的也要学习一下人家的官方"原配"啊。
<!-- more -->
# SpringBoot整合Thymeleaf
## 1. Thymeleaf
### 1.1 模板引擎
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;市面上主流的 Java 模板引擎有：JSP、Velocity、Freemarker、Thymeleaf。JSP 本质也是模板引擎，SpringBoot官方推荐使用**Thymeleaf**模板引擎。模板引擎原理图如下，模板引擎的作用都是将模板(页面)和数据进行整合然后输出显示，区别在于不同的模板使用不同的语法，如JSP的JSTL表达式，以及JSP自己的表达式和语法，同理Thymeleaf也有自己的语法。
![模板引擎](模板引擎.png)

### 1.2 简述Thymeleaf
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thymeleaf是适用于Web和独立环境的现代服务器端Java模板引擎。简单说， Thymeleaf 是一个跟 Velocity、FreeMarker 类似的模板引擎，它可以完全替代JSP。Thymeleaf的主要目标是为开发工作流程带来优雅的自然模板 -HTML可以在浏览器中正确显示，也可以作为静态原型工作，从而可以在开发团队中加强协作。

### 1.3 Thymeleaf优势
+ 1、Thymeleaf 在有网络和无网络的环境下皆可运行，即它可以让美工在浏览器查看页面的静态效果，也可以让程序员在服务器查看带数据的动态页面效果。这是由于它支持 html 原型，然后在 html 标签里增加额外的属性来达到模板+数据的展示方式。浏览器解释 html 时会忽略未定义的标签属性，所以 thymeleaf 的模板可以静态地运行；当有数据返回到页面时，Thymeleaf 标签会动态地替换掉静态内容，使页面动态显示。
+ 2、Thymeleaf 开箱即用的特性。它提供标准和spring标准两种方言，可以直接套用模板实现JSTL、 OGNL表达式效果，避免每天套模板、该jstl、改标签的困扰。同时开发人员也可以扩展和创建自定义的方言。
+ 3、Thymeleaf 提供spring标准方言和一个与 SpringMVC 完美集成的可选模块，可以快速的实现表单绑定、属性编辑器、国际化等功能。

## 2. SpringBoot使用Thymeleaf
### 2.1 配置Thymeleaf
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.在pom.xml文件引入thymeleaf模板引擎依赖。
```xml
<!-- thymeleaf依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.在application.yml中配置thymeleaf模板解析器属性。thymeleaf配置是在spring配置下的。
```yml
spring:
    thymeleaf:
        enabled: true                           # 是否为Web框架启用Thymeleaf视图解析
        prefix: classpath:/templates/           # 配置视图解析器前缀
        suffix: .html                           # 配置试图解析器后缀
        mode: HTML                              # 应用于模板的html模式
        encoding: utf-8                         # 编码格式 
        servlet.content-type: text/html         # 指定请求信息格式
        cache: false                            # 模板缓存。开发时关闭缓存,不然没法看到实时页面
```

### 2.2 使用Thymeleaf
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.在templates文件夹下创建thymeleaf.html模板文件。
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h4>亲爱的<span th:text="${name}"></span>，你好！</h4>  
</body>
</html>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.创建一个controller类。
```java
package com.example.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @Author: fby
 * @Date: 2019/10/24 下午7:53
 * @version 1.0
 */
@Controller
public class ThymeleafController {
    @RequestMapping("/thymeleaf.do")
    public String thymeleaf(Model model){
        model.addAttribute("name","fanfan");
        return "thymeleaf";
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.然后启动项目，在浏览器中输入[localhost:8080/thymeleaf.do](localhost:8080/thymeleaf.do)，成功的页面显示如下。
![测试](测试.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**注意:**这里需要说明一下，html5创建的模板里`<meta>`标签是下面这样的。这个一开始是没有结束符号的，springboot默认使用的版本是`thymeleaf2.0`，如果使用`3.0`的话需要将其改写为带有结束标语的。要么就删掉，因为在`yml`文件中已经设置了编码(一般不建议)。
```html
html5模板：
<meta charset="UTF-8">
thymeleaf3.0版本改写为
<meta charset="UTF-8" />
```

## 3. Thymeleaf详解
### 3.1 引入
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要想在html文件里使用thymeleaf的语法，首先要现在文件里引入`th`标签的命名空间。`xmlns`属性可以在文档里定义一个或多个可供选择的命名空间。
```html
<html xmlns="http://www.w3.org/1999/xhtml">
```

### 3.2 常用th标签
```html
关键字          功能介绍                                           示例
th:id           替换id                              <input th:id="'xxx' + ${collect.id}"/>
th:text         文本替换                            <p th:text="${collect.description}">description</p>
th:utext        支持html的文本替换                  <p th:utext="${htmlcontent}">conten</p>
th:object       替换对象                            <div th:object="${session.user}">
th:value        属性赋值                            <input th:value="${user.name}" />
th:with         变量赋值运算                         <div th:with="isEven=${prodStat.count}%2==0"></div>
th:style        设置样式                            <th:style="'display:' + @{(${sitrue} ? 'none' : 'inline-block')} + ''">
th:onclick      点击事件                            <th:οnclick="'getCollect()'">
th:each         属性赋值                            <tr th:each="user,userStat:${users}">
th:if           判断条件                            <a th:if="${userId == collect.userId}" >
th:unless       和th:if判断相反                     <a th:href="@{/login}" th:unless=${session.user != null}>Login</a>
th:href         链接地址                            <a th:href="@{/login}" th:unless=${session.user != null}>Login</a> />
th:switch       多路选择 配合th:case 使用            <div th:switch="${user.role}">
th:case         th:switch的一个分支                  <p th:case="'admin'">User is an administrator</p>
th:fragment     布局标签，定义一个代码片段，          <div th:fragment="alert">
                方便其它地方引用   
th:include      布局标签，替换内容到引入的文件        <head th:include="layout :: htmlhead" th:with="title='xx'"></head> />
th:replace      布局标签，替换整个标签到引入的文件     <div th:replace="fragments/header :: title"></div>
th:selected     selected选择框 选中                  th:selected="(${xxx.id} == ${configObj.dd})"
th:src          图片类地址引入                       <img class="img-responsive" alt="App Logo" th:src="@{/img/logo.png}" />
th:inline       定义js脚本可以使用变量               <script type="text/javascript" th:inline="javascript">
th:action       表单提交的地址                       <form action="subscribe.html" th:action="@{/subscribe}">
th:attr         设置标签属性，多个属性可以用比如       <th:attr="src=@{/image/aa.jpg},title=#{logo}">
                逗号分隔 
th:remove       删除某个属性                         <tr th:remove="all">
　　　　　　　　　　　　　　　　　　　　			   1.all:删除包含标签和所有的孩子。
　　　　　　　　　　　　　　　　　　　　			   2.body:不包含标记删除,但删除其所有的孩子。
　　　　　　　　　　　　　　　　　　　　			   3.tag:包含标记的删除,但不删除它的孩子。
　　　　　　　　　　　　　　　　　　　　			   4.all-but-first:删除所有包含标签的孩子,除了第一个。
　　　　　　　　　　　　　　　　　　　　			   5.none:什么也不做。这个值是有用的动态评估。
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;还有非常多的标签，这里只列出最常用的几个,由于一个标签内可以包含多个th:x属性，其生效的优先级顺序为: 
include,each,if/unless/switch/case,with,attr/attrprepend/attrappend,value/href,src ,etc,text/utext,fragment,remove。

### 3.3 常用语法
#### 3.3.1 标准表达式
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thymeleaf语法有很多，想更深入了解的话可以去看[官方文档](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html)。在这先简单介绍一下标准表达式功能：
+ 简单表达式
   + 变量表达式: **${...}**
   + 选择表达式: **\*{...}**
   + 消息表达式: **#{...}**
   + 链接URL表达式: **@{...}**
+ 文字
   + 文本文字: `'one text'`，...
   + 号码文字: `0`，`34`，`3.2`，...
   + 布尔文字: `true`，`false`
   + 空文字: `null`
   + 文字标记: `one`，`sometext`，...
+ 文字操作
   + 字符串串联: `+`
   + 文字替换: `|The name is ${naem}|`
+ 算数运算
   + 二元运算符: `+`，`-`，`*`，`/`，`%`
   + 减号(一元运算符): `-`
+ 布尔运算
   + 二元运算符: `and`，`or`
   + 布尔否定(一元运算符): `!`，`not`
+ 比较和平等
   + 比较: `>`，`<`，`>=`，`<=` `（gt，lt，ge，le）`
   + 等号运算符: `==`，`!=` `(eq，ne)`
+ 条件运算符
   + 如果-则: `(if) ? (then)`
   + 如果-则-否则: `(if) ? (then) : (else)`
   + 默认: `(value) ?: (defautvalue)`

> 以上所有的这些功能都可以进行组合和嵌套。
> `'User is of type ' + (${user.isAdmin()} ? 'Administrator' : (${user.type} ?: 'Unknown'))`

#### 3.3.2 赋值、字符串的拼接
```html
<!-- 直接取值 -->
<p  th:text="${collect.description}"></p>
<!-- 标签内赋值 -->
<p>hello，[[${collect.description}]]</p>
<!-- 取值拼接 -->
<span th:text="'Welcome to our application, ' + ${user.name} + '!'">
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字符串拼接还有另外一种简洁的写法。
```html
<span th:text="|Welcome to our application, ${user.name}!|">
```

#### 3.3.3 条件判断If/Unless
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thymeleaf中使用`th:if`和`th:unless`属性进行条件判断，下面的例子中，`<a>`标签只有在th:if中条件成立时才显示：
```html
<a th:if="${myself=='yes'}" > </i> </a>
<a th:unless=${session.user != null} th:href="@{/login}" >Login</a>
```
> th:unless于th:if恰好相反，只有表达式中的条件不成立，才会显示其内容。
也可以使用 (if) ? (then) : (else) 这种语法来判断显示的内容。

#### 3.3.4 for循环
```html
  <tr  th:each="item,iterStat : ${list}"> 
     <th scope="row" th:text="${collect.id}">1</th>
     <td >
        <img th:src="${item.webLogo}"/>
     </td>
     <td th:text="${item.url}">Mark</td>
     <td th:text="${item.title}">Otto</td>
     <td th:text="${item.description}">@mdo</td>
     <td th:text="${item.index}">index</td>
  </tr>
```
> list是后台传入的数据，item是设置每次循环的对象名(自定义)。

iterStat称作状态变量，属性有:
+ index: 当前迭代对象的index(从0开始计算)
+ count: 当前迭代对象的index(从1开始计算)
+ size: 被迭代对象的大小
+ current: 当前迭代变量
+ even/odd: 布尔值，当前循环是否是偶数/奇数(从0开始计算)
+ first: 布尔值，当前循环是否是第一个
+ last: 布尔值，当前循环是否是最后一个

#### 3.3.5 URL
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;URL在Web应用模板中占据着十分重要的地位，需要特别注意的是Thymeleaf对于URL的处理是通过语法 **@{…}**来处理的。如果需要Thymeleaf对URL进行渲染，那么务必使用`th:href`，`th:src`等属性，也可以用来引入css、js、图片等文件。
```html
<!-- 超链接 -->
<a th:href="@{/order/{orderId}/details(orderId=${o.id})}">
<!-- 引入css -->
<link th:href="@{/css/bootstrap.min.css}" rel="stylesheet">
<!-- 设置背景 -->
<div th:style="'background:url(' + @{/<path-to-image>} + ');'"></div>
<!-- 根据属性值改变背景 -->
 <div class="media-object resource-card-image"  th:style="'background:url(' + @{(${collect.webLogo}=='' ? 'img/favicon.png' : ${collect.webLogo})} + ')'" ></div>
```
几点说明：
+ 上列中URL最后的(orderId=${o.id})表示将括号内的内容作为URL参数处理，该语法避免使用字符串拼接，大大提高了可读性。
+ @{...}表达式中可以通过{orderId}访问COntext中的orderId变量。
+ @{/order}是Context相关的相对路径，在渲染时会自动添加上当前Web应用的Context名字。例如contest名字为app，那么解析的结果就是/app/order。

#### 3.3.6 内联JS
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;内联文本：**[[...]]**内联文本的表示方式，**[[…]]**之间的内容可以被赋值。为了使其生效，必须在此标签或者任何父标签上有**th:inline**属性。此属性有三种值(`text`、`javascript`、`none`)。**th:inline**也可以在父标签上使用，比如作为body上的标签。表达式在javascript中使用时，先用属性声明一下：th:inline="javascript"，然后我们开始在js中声明变量：
```javascript
<script th:inline="javascript">
    /*<![CDATA[*/
        ...
        var username = /*[[${session.user.name}]]*/ 'Sebastian';
        ...
    /*]]>*/
</script>
```
**/\*[[...]]\*/**表达式的理解如下;
+ /\*...\*/中的内容子啊浏览器打开静态网页时会被忽略。
+ 'Sebastian' 会在浏览器中显示。(静态时)。
+ Thymeleaf解析时，会解析/\*[[…]]\*/的内容，并把获得的值替换掉/\*[[…]]\*/后面的内容。

所以执行的结果如下：
```javascript
<script th:inline="javascript">
    /*<![CDATA[*/
        ...
        var username = 'John Apricot';
        ...
    /*]]>*/
</script>
```
当然，你也可以不用注释，就像下面这样：
```javascript
<script th:inline="javascript">
    /*<![CDATA[*/
        ...
        var username = [[${session.user.name}]];
        ...
    /*]]>*/
</script>
```
这会让它在静态显示时出现错误。
>注意：引擎求值后注入式是智能的，它可以赋值以下类型的数据：`String`、`Numbers`、`Booleans`、`Arrays`、`Collections`、`Maps`、`Beans (objects with getter and setter methods)`。

举个栗子：
```javascript
script th:inline="javascript">
    /*<![CDATA[*/
    ...
    var user = /*[[${session.user}]]*/ null;
    ...
    /*]]>*/
</script>
```
${session.user}会获取一个user对象。写入如下：
```javascript
<script th:inline="javascript">
    /*<![CDATA[*/
    ...
    var user = {'age':null,'firstName':'John','lastName':'Apricot',
    'name':'John Apricot','nationality':'Antarctica'};
    ...
    /*]]>*/
</script>
```

引擎同样允许增加和删除代码块。增加代码块：
```javascript
var x = 23;
/*[+
var msg = 'This is a working application';
+]*/
var f = function() {
...
```
解析如下：
```javascript
var x = 23;
var msg = 'This is a working application';
var f = function() {
...
```
删除代码块：
```javascript
var x = 23;
/*[- */
var msg = 'This is a non-working template';
/* -]*/
var f = function(){
...
```
解析如下：
```javascript
var x = 23;
var f = function(){
...
```
>增加和删除代码块只是有这一块的知识点，所以暂时先写上了，但是具体是干啥的，我也不清楚，反正我是没用到。

#### 3.3.7 信息表达式
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先举个简单的栗子:
```javascript
<p th:utext="#{home.welcome}">Welcome to our grocery store!</p>
```
其中`home.welcome=欢迎光临本店`。
如果消息文本不完全是静态的会发生什么？有时候我们需要在消息中增加变量，比如输出访问者的名字怎么办？可以这样办：
```javascript
<p th:utext="#{home.welcome(${session.user.name})}">
Bienvenido a nuestra tienda de comestibles, 木鱼!
</p>
```
其中`home.welcome=欢迎光临本店, {0}!`
在这里，参数可以是字符型也可是树数值型或者日期型。当然如果我们需要多个参数的话，类推即可，并且我们也可以内嵌表达式替换字符串，比如：
```javascript
<p th:utext="#{${welcomeMsgKey}(${session.user.name})}">
Welcome to our grocery store, 木鱼!
</p>
```

#### 3.3.8 选择表达式
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;变量不仅能用在#{ }上，还能用在* { }上。两者的区别在于* { }上的的变量首先是选定对象的变量。如果不选定对象，那么是整个上下文环境中的变量和#{ }相同。选择对象用什么呢?`th:object`标签属性。我们使用它在我们的用户配置文件(userprofile.html)页面:
```javascript
<div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```
这个语法等同于以下：
```javascript
<div>
    <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p>
</div>
```
当然了，这两种用法是可以混合的:
```javascript
<div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```
如果一个对象已经被选择，即th:object=”${session.user}”。那么我们也可以使用#object对象去引用。
```javascript
<div th:object="${session.user}">
    <p>Name: <span th:text="${#object.firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```
就像之前说的，如果没有对象被选中，那么`#{}`和`*{}`表达式的意义是相同的。
```javascript
<div>
    <p>Name: <span th:text="*{session.user.name}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{session.user.surname}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{session.user.nationality}">Saturn</span>.</p>
</div>
```

### 3.4 基本对象
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在上下文变量上评估OGNL表达式时，某些对象可用于表达式，以提高灵活性。这些对象（根据OGNL标准）将以#符号开头进行引用。
#### 3.4.1 基础对象
+ **#ctx**: 上下文对象。
+ **#vars**: 上下文变量。
+ **#locale**: 上下文语言环境。
+ **#httpSession**: HttpSession对象(仅在Web上下文中)。
+ **#httpServletRequest**: HttpServletRequest对象(仅在Web上下文中)。

可以通过以下方式引用：
```javascript
Established locale country: <span th:text="${#locale.country}">US</span>.
```

#### 3.4.2 请求/会话属性等Web上下文名称空间
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Web环境中使用Thymeleaf时，我们可以使用一系列快捷方式来访问请求参数，会话属性和应用程序属性。
>请注意，这些不是上下文对象，而是作为变量添加到上下文中的映射，因此我们不使用即可访问它们#。因此，它们以某种方式充当命名空间。

+ **param**: 用于检索请求参数。${param.foo}是String[]带有foorequest参数值的a ，因此${param.foo[0]}通常用于获取第一个值。
+ **session**: 用于获取会话属性。例如`${session.user.name}`。
+ **application**: 用于检索应用程序/ servlet上下文属性。

#### 3.4.3 Web上下文对象
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Web环境中，还可以直接访问以下对象(请注意，这些是对象，而不是映射/命名空间)。
+ **#httpServletRequest**: 直接访问`javax.servlet.http.HttpServletRequest`与当前请求关联的对象。
+ **#httpSession**: 直接访问`javax.servlet.http.HttpSession`与当前请求关联的对象。

> 这里只是简单说一下这些对象，因为我也没有用过，想要深入使用的可以在[官网附录A](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#appendix-a-expression-basic-objects)中阅读这些对象的完整参考。 

### 3.5 Thymeleaf对象
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了这些基本的对象,Thymeleaf将为我们提供一套实用的对象。来帮助我们我们执行常见的任务。
+ **#dates** : 为 java.util.Date对象提供工具方法,比如：格式化,提取年月日等.
+ **#calendars** : 类似于#dates , 但是只针对java.util.Calendar对象.
+ **#numbers** : 为数值型对象提供工具方法。
+ **#strings** :为String 对象提供工具方法。如: contains, startsWith, prepending/appending等。
+ **#objects** : 为object 对象提供常用的工具方法。
+ **#bools** : 为boolean 对象提供常用的工具方法。
+ **#arrays** : 为arrays 对象提供常用的工具方法。
+ **#lists** :为lists对象提供常用的工具方法。
+ **#sets** : 为sets对象提供常用的工具方法。
+ **#maps** : 为maps对象提供常用的工具方法。
+ **#aggregates** :为创造一个arrays 或者 collections聚集函数提供常用的工具方法。
+ **#messages** : 用于获取变量表达式内的外部化消息，其方式与使用`#{...}`语法获得消息的方式相同。
+ **#ids** : 处理id可能重复的属性的实用方法(例如，由于迭代的结果)。

> 这里只是简单说一下这些对象，因为我也没有用过，想要深入使用的可以在[官网附录B](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#appendix-b-expression-utility-objects)中阅读这些对象的完整参考。

## 4 总结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在写这篇博客之前关于Thyemleaf模板引擎我也就用过一次，踩过的坑也不少，所以写这篇博客时才各种百度、官网文档等等的去搜集相关知识，整理个也算比较详细了。用的比较多的那些给的都有示例代码，其他的先暂时当作了解，日后开发项目有需要了再来过看。