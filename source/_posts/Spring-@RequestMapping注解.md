---
title: Spring-@RequestMapping注解
route: spring-RequestMapping
date: 2019-9-1 18:00:32
tags: [Spring,Java]
categories: Spring
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;@RequestMapping 是 Spring Web 应用程序中最常被用到的注解之一。这个注解会将 HTTP 请求映射到 MVC 和 REST 控制器的处理方法上。
<!-- more -->
# @RequestMapping注解
## 1. 基础用法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;@RequestMapping是配置web请求的映射，该注解可以在控制器类的级别和/或其中的方法的级别上使用。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在类的级别上的注解会将一个特定请求或者请求模式映射到一个控制器之上。之后还可以另外添加方法级别的注解来进一步值定到处理方法的映射关系。
下面是一个同时在类和方法上应用了@RequestMapping注解示例：
```java
@RestController  
@RequestMapping("/home")  
public class IndexController {  
    @RequestMapping("/")  
    String get() {  
        //mapped to hostname:port/home/  
        return "Hello from get";  
    }  
    @RequestMapping("/index")  
    String index() {  
        //mapped to hostname:port/home/index/  
        return "Hello from index";  
    }  
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如上述代码所示，到/home的请求会由get()方法来处理，而到/home/index的请求会有index()方法处理。

## 2. 处理多个URL
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以将多个请求映射到一个方法上面，只需要添加一个带有请求路径值列表的@RequestMapping注解就行了。
```java
@RestController  
@RequestMapping("/home")  
public class IndexController {  
  
    @RequestMapping(value = {  
        "",  
        "/page",  
        "page*",  
        "view/*,**/msg"  
    })  
    String indexMultipleMapping() {  
        return "Hello from index multiple mapping.";  
    }  
}  
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;@RequestMapping支持通配符以及ANT风格的路径。上述示例代码中，以下的的URL都会由indexMultipleMapping()来处理:
```js
localhost:8080/home
localhost:8080/home/
localhost:8080/home/page
localhost:8080/home/pageabc
localhost:8080/home/view/
localhost:8080/home/view/view
```

## 3. 处理动态URI 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;@RequestMapping 注解可以同 @PathVaraible 注解一起使用，用来处理动态的 URI，URI 的值可以作为控制器中处理方法的参数。你也可以使用正则表达式来只处理可以匹配到正则表达式的动态 URI。 
```java
@RestController  
@RequestMapping("/home")  
public class IndexController {  
    @RequestMapping(value = "/fetch/{id}", method = RequestMethod.GET)  
    String getDynamicUriValue(@PathVariable String id) {  
        System.out.println("ID is " + id);  
        return "Dynamic URI parameter fetched";  
    }  
    @RequestMapping(value = "/fetch/{id:[a-z]+}/{name}", method = RequestMethod.GET)  
    String getDynamicUriValueRegex(@PathVariable("name") String name) {  
        System.out.println("Name is " + name);  
        return "Dynamic URI parameter fetched using regex";  
    }  
} 
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在这段代码中，方法 getDynamicUriValue() 会在发起到localhost:8080/home/fetch/10 的请求时执行。这里 getDynamicUriValue() 方法 id 参数也会动态地被填充为 10 这个值。 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;方法 getDynamicUriValueRegex() 会在发起到localhost:8080/home/fetch/category/shirt 的请求时执行。不过，如果发起的请求是 /home/fetch/10/shirt 的话，会抛出异常，因为这个URI并不能匹配正则表达式。 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;@PathVariable 同 @RequestParam的运行方式不同。你使用 @PathVariable 是为了从 URI 里取到查询参数值。换言之，你使用 @RequestParam 是为了从 URI 模板中获取参数值。 

## 4. 带有@RequestParam参数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;@RequestParam 注解配合 @RequestMapping 一起使用，可以将请求的参数同处理方法的参数绑定在一起。 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;@RequestParam 注解使用的时候可以有一个值，也可以没有值。这个值指定了需要被映射到处理方法参数的请求参数, 代码如下所示：
```java
@RestController 
@RequestMapping("/home")  
public class IndexController {   
    @RequestMapping(value = "/id")  
    String getIdByValue(@RequestParam("id") String personId) {  
        System.out.println("ID is " + personId);  
        return "Get ID from query string of URL with value element";  
    }  
    @RequestMapping(value = "/personId")  
    String getId(@RequestParam String personId) {  
        System.out.println("ID is " + personId);  
        return "Get ID from query string of URL without value element";  
    }  
}  
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在代码的第5行，id这个请求参数被映射到了getIdByValue() 这个处理方法的参数 personId 上。 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果请求参数和处理方法参数的名称一样的话，@RequestParam注解的value这个参数就可以省略掉了，如代码的第10行所示。

@RequestParam注解的**required**这个参数定义了参数值是否是必须要传的。
```java
@RestController  
@RequestMapping("/home")  
public class IndexController {  
    @RequestMapping(value = "/name")  
    String getName(@RequestParam(value = "person", required = false) String personName) {  
        return "Required element of request param";  
    }  
}  
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在这段代码中，因为 required 被指定为 false，所以 getName() 处理方法对于如下两个 URL 都会进行处理：
>/home/name/person=xyz
>/home/name

@RequestParam的**defaultValue**取值就是用来给取值为空的请求参数提供一个默认值的。
```java
@RestController  
@RequestMapping("/home")  
public class IndexController {  
    @RequestMapping(value = "/name")  
    String getName(@RequestParam(value = "person", defaultValue = "John") String personName) {  
        return "Required element of request param";  
    }  
}  
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在这段代码中，如果person这个请求参数为空，那么getName()处理方法就会接收"John"这个默认值作为其参数；如果person请求参数不为空，那么方法接收的personName参数就是person。

## 5. 处理请求参数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;@RequestMapping直径二的params元素可以进一步缩小请求映射的定位范围。使用params元素，可以让多个处理方法处理到同一个URL的请求，而这些请求是不一样的。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以用`myParams=myValue`这种格式来定义参数，也可以使用通配符来指定特定的参数值在请求中是不是受支持的。
```java
@RestController  
@RequestMapping("/home")  
public class IndexController {  
    @RequestMapping(value = "/fetch", params = {  
        "personId=10"  
    })  
    String getParams(@RequestParam("personId") String id) {  
        return "Fetched parameter using params attribute = " + id;  
    }  
    @RequestMapping(value = "/fetch", params = {  
        "personId=20"  
    })  
    String getParamsDifferent(@RequestParam("personId") String id) {  
        return "Fetched parameter using params attribute = " + id;  
    }  
}  
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在这段代码中，getParams() 和 getParamsDifferent() 两个方法都能处理相同的一个 URL (/home/fetch) ，但是会根据 params 元素的配置不同而决定具体来执行哪一个方法。 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;例如，当 URL 是 /home/fetch?id=10 的时候, getParams() 会执行，因为 id 的值是10,。对于 localhost:8080/home/fetch?personId=20 这个URL, getParamsDifferent() 处理方法会得到执行，因为 id 值是 20。 

## 6. 处理HTTP得各种方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Spring MVC的@RequestMapping注解能够处理 HTTP 请求的方法, 比如 GET, PUT, POST, DELETE 以及 PATCH。所有的请求默认都会是 HTTP\GET 类型的。 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了能将一个请求映射到一个特定的 HTTP 方法，你需要在 @RequestMapping 中使用**method**来声明 HTTP 请求所使用的方法类型，如下所示： 
```java 
@RestController  
@RequestMapping("/home")  
public class IndexController {  
    @RequestMapping(method = RequestMethod.GET)  
    String get() {  
        return "Hello from get";  
    }  
    @RequestMapping(method = RequestMethod.DELETE)  
    String delete() {  
        return "Hello from delete";  
    }  
    @RequestMapping(method = RequestMethod.POST)  
    String post() {  
        return "Hello from post";  
    }  
    @RequestMapping(method = RequestMethod.PUT)  
    String put() {  
        return "Hello from put";  
    }  
    @RequestMapping(method = RequestMethod.PATCH)  
    String patch() {  
        return "Hello from patch";  
    }  
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面得代码中，@RequestMapping注解中的method元素声明了HTTP请求方法的HTTP请求类型。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所有的请求处理方法都会处理从"/home"这个同一URL进来的请求，但是要看指定的HTTP方法是什么来决定用哪个方法来处理。例如一个POST类型的请求/home会交给post()方法来处理。

## 7. 默认的处理方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在控制器中，你可以有一个默认的处理方法，它可以在有一个向默认URL发起请求时被执行。下面时默认处理方法的示例。
```java
@RestController  
@RequestMapping("/home")  
public class IndexController {  
    @RequestMapping()  
    String  
    default () {  
        return "This is a default method for the class";  
    }  
}  
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在这段代码中，向/home发起的一个请求将会有default()来处理，因为注解没有指向任何值。

## 8. 快捷方式
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Spring 4.3 引入了方法级注解的变体，也被叫做 @RequestMapping 的组合注解。组合注解可以更好的表达被注解方法的语义。它们所扮演的角色就是针对 @RequestMapping 的封装，而且成了定义端点的标准方法。 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;例如，@GetMapping 是一个组合注解，它所扮演的是 @RequestMapping(method =RequestMethod.GET) 的一个快捷方式。 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;方法级别的注解变体有如下几个：
+ @GetMapping
+ @PostMapping
+ @PutMapping
+ @DeleteMapping
+ @PatchMapping
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如下代码展示了如何使用组合注解。
```java
@RestController  
@RequestMapping("/home")  
public class IndexController {  
    @GetMapping("/person")  
    public @ResponseBody ResponseEntity < String > getPerson() {  
        return new ResponseEntity < String > ("Response from GET", HttpStatus.OK);  
    }  
    @GetMapping("/person/{id}")  
    public @ResponseBody ResponseEntity < String > getPersonById(@PathVariable String id) {  
        return new ResponseEntity < String > ("Response from GET with id " + id, HttpStatus.OK);  
    }  
    @PostMapping("/person")  
    public @ResponseBody ResponseEntity < String > postPerson() {  
        return new ResponseEntity < String > ("Response from POST method", HttpStatus.OK);  
    }  
    @PutMapping("/person")  
    public @ResponseBody ResponseEntity < String > putPerson() {  
        return new ResponseEntity < String > ("Response from PUT method", HttpStatus.OK);  
    }  
    @DeleteMapping("/person")  
    public @ResponseBody ResponseEntity < String > deletePerson() {  
        return new ResponseEntity < String > ("Response from DELETE method", HttpStatus.OK);  
    }  
    @PatchMapping("/person")  
    public @ResponseBody ResponseEntity < String > patchPerson() {  
        return new ResponseEntity < String > ("Response from PATCH method", HttpStatus.OK);  
    }  
}  
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在这段代码中，每一个处理方法都使用 @RequestMapping 的组合变体进行了注解。尽管每个变体都可以使用带有方法属性的 @RequestMapping 注解来互换实现, 但组合变体仍然是一种最佳的实践 — 这主要是因为组合注解减少了在应用程序上要配置的元数据，并且代码也更易读。 
