---
title: SpringBoot-整合SpringDataJPA
route: springboot-springDataJPA
date: 2019-10-27 10:36:04
tags: [SpringBoot,Java,Spring]
categories: SpringBoot
image: /images/cover/springboot-springDataJPA.png
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JPA的全称是**Java Persistence API**，Persistence 是持久化的意思。所以，中文全称是【JAVA对象持久化的 API】。简单来说，可以理解为是一种JAVA的标准规范，这个规范为JAVA对象的持久化制定了一些标准的接口。要注意的是，JPA只是一个接口规范，而不是实现。具体实现由各供应商来完成，例如Hibernate，TopLink,OpenJPA都很好地实现了JPA接口。
<!-- more -->
# SpringBoot整合SpringDataJPA
## 1. SpringBootDataJPA
### 1.1 简述
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpringDataJPA是较大的SpringData系列的一部分，可轻松实现基于JPA的存储库。该模块处理对基于JPA的数据访问层的增强支持。它使构建使用数据访问技术的Spring支持的应用程序变得更加容易。默认底层是Hibernate，使用JPA的Repository能极大的减少对数据库的访问的代码量，仅仅使用内部接口就可以完成简单的CRUD等操作。

### 1.2 特征
+ 基于Spring和JPA构建存储库的先进支持。
+ 支持Querydsl谓词，从而支持类型安全的JPA查询。
+ 域类的透明审核。
+ 分页支持，动态查询支持，集成自定义数据访问代码的能力。
+ `@Query`引导时验证带注释的查询。
+ 支持基于XML的实体映射。
+ 通过引入JavaConfig的存储库配置`@EnableJpaReposituries`。

## 2. 搭建项目
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该项目是一个完整的开发项目，所有的逻辑代码都放在了Controller层，数据源使用alibaba的*Druid*数据源。
### 2.1 导入依赖/修改配置文件
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先在pom.xml文件原来的基础中添加依赖。
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.12</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后再修改application配置文件，添加关于Druid和SpringBootJPA的依赖。
```yml
spring:
  # 配置视图解析
  mvc:
    view:
      prefix: /WEB-INF/views/
      suffix: .jsp
  # 配置数据源
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://127.0.0.1:3306/test?characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: admin
    driver-class-name: com.mysql.cj.jdbc.Driver
    #最大活跃数
    maxaActive: 20
    #初始化数量
    InitialSize: 1
    #最大连接等待时间
    maxWait: 60000
    #打开PSCache，并指定大小
    poolPreparedStatements: true
    maxPoolPreparedStatementPerConnectionSize: 20
    minIdle: 1
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: select 1 from dual
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    filters: stat, wall, log4j
  # 配置JPA
  jpa:
    database: mysql
    show-sql: true                 # 是否打印sql
    generate-ddl: true             # 是否生成ddl
    hibernate:                     # 数据库表的创建方式:更新 
        ddl-auto: update
    properties:
        ### 数据库方言,告诉hibernate这是mysql
        hibernate.dialect: com.demo.mysql.MySQLDialectUTF8
        ### 控制条打印sql格式化输出
        hibernate.format_sql: true
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;数据库方言设置，新建`mysql`包，创建`MySQLDialectUTF8`类。将默认的字符集编码设置为UTF8。
```java
package com.example.demo.mysql;

import org.hibernate.dialect.MySQL5Dialect;

/**
 * 重写数据库方言，设置默认字符集为utf8
 */
public class MySQLDialectUTF8 extends MySQL5Dialect {

    @Override
    public String getTableTypeString() {
        return " ENGINE=InnoDB DEFAULT CHARSET=utf8";
    }
}
```
**注意**
配置数据库连接，8.0以上的版本再写法上有些不一样，以下是新版写法。8.0以下版本的写法不变。
```yml
url: jdbc:mysql://127.0.0.1:3306/test?characterEncoding=UTF-8&serverTimezone=UTC
driver-class-name: com.mysql.cj.jdbc.Driver
```
这时候mysql驱动也需要是8.0以上版本的。pom.xml依赖如下。
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.16</version>
</dependency>
```

### 2.2 使用JpaRepository
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为JPA的底层就是Hibernate，所以需要一个实体类对数据库表表结构进行映射，在启动项目时，会自动根据实体类创建相应的表结构。
```java
package com.example.demo.domain;

import javax.persistence.*;
import java.io.Serializable;

/**
 * @Author: FBY
 * @Date: 2019/10/27 14:26
 * @Version 1.0
 */
@Entity
@Table(name = "stu")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Integer id;
    
    @Column(name = "name")
    private String name;
    
    @Column(name = "age")
    private Integer age;
    
    @Column(name = "address")
    private String address;

    /*getter and setter*/    
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面对该类中用到的注解做一个简单说明：使用`@Entity`会对实体类进行持久化操作，当JPA检测到实体类中有`@Entity`注解时，会在数据库中生成相对应的表结构信息。`@Table`用来指定该实体类对应的表明。`@Id`用来指定主键，配合`@GeneratedValue(strategy = GenerationType.IDENTITY)`指定主键的自增策略，这里将主键自增交给数据库去做，所以使用`IDENTITY`。`@Column`用来指定对应表中的字段名。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之后创建一个`jpa`包，然后在下面创建`StudentJPA`接口，继承**JpaRepository**，需要两个参数，一个时实体类对象，一个是主键类型。
```java
package jpa;

import com.example.demo.domain.Student;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * @Author: FBY
 * @Date: 2019/10/27 14:53
 * @Version 1.0
 */
public interface StudentJPA extends JpaRepository<Student,Integer> {
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看`JpaRepository`源码可以之后该接口又继承了`PagingAndSortingRepository`和`QueryByExampleExecutor`这两个接口，`PagingAndSortingRepository`又继承了`CrudRepository`接口。这些接口基本上看名字就知道这个接口大概实现了什么方法，这就是命名规范的好处啊。

#### 2.2.1 CrudRepository
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;看名字可以知道，该接口包含饿了crud等操作，也就是`creat`、`select`、`delete`、`update`、`exist`、`count`。如果继承了该接口，就会拥有该接口所有的实现。
```java
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    <S extends T> S save(S var1);
    <S extends T> Iterable<S> saveAll(Iterable<S> var1);
    Optional<T> findById(ID var1);
    boolean existsById(ID var1);
    Iterable<T> findAll();
    Iterable<T> findAllById(Iterable<ID> var1);
    long count();
    void deleteById(ID var1);
    void delete(T var1);
    void deleteAll(Iterable<? extends T> var1);
    void deleteAll();
}
```

#### 2.2.2 PagingAndSortingRepository
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该接口时分页和排序，，而且继承了`CrudRepository`接口，拥有其所有的接口实现。
```java
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
    Iterable<T> findAll(Sort var1);
    Page<T> findAll(Pageable var1);
}
```

#### 2.2.3 QueryByExampleExecutor
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个接口实现了条件查询和复杂查询，可以使用exmple的方式查询。
```java
public interface QueryByExampleExecutor<T> {
    <S extends T> Optional<S> findOne(Example<S> var1);
    <S extends T> Iterable<S> findAll(Example<S> var1);
    <S extends T> Iterable<S> findAll(Example<S> var1, Sort var2);
    <S extends T> Page<S> findAll(Example<S> var1, Pageable var2);
    <S extends T> long count(Example<S> var1);
    <S extends T> boolean exists(Example<S> var1);
}
```

#### 2.2.4 JpaRepository
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们用的就是这个接口，它拥有以上所有接口的方法实现，并且添加了条件查询和保存集合数据的方法，实现了该接口基本上简单的数据库操作就不需要我们自己写SQL语句了。
```java
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    List<T> findAll();
    List<T> findAll(Sort var1);
    List<T> findAllById(Iterable<ID> var1);
    <S extends T> List<S> saveAll(Iterable<S> var1);
    void flush();
    <S extends T> S saveAndFlush(S var1);
    void deleteInBatch(Iterable<T> var1);
    void deleteAllInBatch();
    T getOne(ID var1);
    <S extends T> List<S> findAll(Example<S> var1);
    <S extends T> List<S> findAll(Example<S> var1, Sort var2);
}
```

## 3. 使用JPA
### 3.1 创建Service层
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;新建`jpa`包，在该包下创建`StudentuJpa`接口，并让其继承`JpaRepository`，这样该接口就拥有了它的所有方法实现。
```java
package com.example.demo.jpa;

import com.example.demo.domain.Student;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * @Author: FBY
 * @Date: 2019/10/27 16:06
 * @Version 1.0
 */
public interface StudentJpa extends JpaRepository<Student,Integer> {
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;新建`service`包，在该包`StudentService`接口。
```java
package com.example.demo.service;

import com.example.demo.domain.Student;
import java.util.List;

/**
 * @Author: FBY
 * @Date: 2019/10/27 16:03
 * @Version 1.0
 */
public interface StudentService {
    // 增加/修改
    void save(Student student);
    // 删除
    void deleteById(Integer id);
    // id查询
    Student findById(Integer id);
    // 查询所有
    List<Student> findAll();
}

```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在`service`包下新建`Impl`包，在该包下创建`StudentServiceImpl`实现类。注入`StudentJpa`。
```java
package com.example.demo.service.Impl;

import com.example.demo.domain.Student;
import com.example.demo.jpa.StudentJpa;
import com.example.demo.service.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

/**
 * @Author: FBY
 * @Date: 2019/10/27 16:04
 * @Version 1.0
 */
@Service
public class StudentServiceImpl implements StudentService {

    @Autowired
    private StudentJpa studentJpa;

    @Override
    public void save(Student student) {
        studentJpa.save(student);
    }

    @Override
    public void deleteById(Integer id) {
        studentJpa.deleteById(id);
    }

    @Override
    public Student findById(Integer id) {
        Optional<Student> optional = studentJpa.findById(id);
         if(optional.isPresent()){
            return optional.get();
        }
        return null;
    }

    @Override
    public List<Student> findAll() {
        return studentJpa.findAll();
    }
}
```
> 关于根据主键进行查找，`findById(Integer id)`返回封装后的对象`Optional<T>`，在Optional类中有很多内置的方法，其中`isPresen()`方法返回Optional对象是否为null的结果，如果当前对象有值就返回`true`，否则返回`false`，当结果有值时，然后调用它的`get()`方法，会返回一个<T>类型的实体类对象，即我们要查询的对象。

> 根据主键查找提供的还有另一个方法，就是`getOne(Integer id)`，这个方法返回的时代理对象，无法直接操作，还有可能会出现`hibernate lazyxxx  no session`的错误，在测试方法上加上`@Transactional`注解可以解决报错的问题。

### 3.2 创建Controller层
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在`controller`包下创建`StudentJPAController`类，这次测试就不再使用页面了，只需要看到返回数据即可，所以注解使用`@RestControlle`，并注入`StudentService`。返回json格式验证数据。
```java
package com.example.demo.controller;

import com.example.demo.service.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author: FBY
 * @Date: 2019/10/27 21:34
 * @Version 1.0
 */
@RestController
public class StudentJPAController {
    @Autowired
    private StudentService service;
}
```

### 3.3 测试
#### 3.3.1 增加/修改
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;往数据库里添加数据只需要将实体类当作参数，调用JPA的`save`方法即可。
```java
@RequestMapping(value = "/add")
public String add(){
    Student stu = new Student();
    stu.setName("fanfan");
    stu.setAge(19);
    stu.setAddress("郑州轻工业大学");
    service.save(stu);
    return "添加成功";
}
```
>save方法不仅仅用于增加，如果传入的实体类中设置了主键，那么save方法就会变为根据主键更新数据库的操作。要注意的是save用于更新时，更新的是实体类里的所有字段，不设置值的字段会被更新成null。

#### 3.3.2 删除
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`JpaRepository`提供的有根据主键删除的方法`deleteById`，直接在底层调用即可。
```java
@RequestMapping(value = "/delete.do")
public String delete(Integer id){
    service.deleteById(id);
    return "删除成功";
}
```

#### 3.3.4 查询
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查询全部直接在底层使用JpaRepository内部实现的`findAll`方法。在浏览器进行访问就可以看到数据库准备的数据。
```java
@RequestMapping(value = "/findAll.do")
public List<Student> findAll(){
    return service.findAll();
}
```
![测试查询所有](测试查询所有.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据id查询一个也是调用底层方法`findById`就可以实现，具体上面已经详细介绍去了，在这里就不演示了。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于实例查询,需要用到这个**ExampleMatcher**------匹配器。
> **ExampleMatcher**实例查询三要素：
> + 实体对象：在ORM框架中与Table对应的域对象，一个对象代表数据库表中的一条记录，如上例中User对象，对应user表。在构建查询条件时，一个实体对象代表的是查询条件中的“数值”部分。如：要查询姓“X”的客户，实体对象只需要存储条件值“X”。
> + ExampleMatcher对象：它是匹配“实体对象”的，表示了如何使用“实体对象”中的“值”进行查询，它代表的是“查询方式”，解释了如何去查的问题。如：要查询姓“X”的客户，即姓名以“X”开头的客户，该对象就表示了“以某某开头的”这个查询方式，如上例中:withMatcher(“userName”, GenericPropertyMatchers.startsWith())
> + 实例：即Example对象，代表的是完整的查询条件。由实体对象（查询条件值）和匹配器（查询方式）共同创建。最终根据实例来findAll即可。

示例：根据姓名、年龄、地址模糊查询。首先在`StudentService`接口里添加方法。
```java
    // 实例查询
    List<Student> findByExample(Student student);
```
然后在`StudentServiceImpl`类中添加实现。
```java
@Override
public List<Student> findByExample(Student student) {
    // 创建匹配器，即如何使用查询条件
    ExampleMatcher matcher = ExampleMatcher.matching()      //构建对象
            .withMatcher("name",ExampleMatcher.GenericPropertyMatchers.startsWith())       // "姓名"采用模糊查询匹配开头，即{name}%
            .withMatcher("age",ExampleMatcher.GenericPropertyMatchers.contains())       // "年龄"采用模糊查询，即%{age}%
            .withMatcher("address",ExampleMatcher.GenericPropertyMatchers.contains())  // "地址"采用模糊查询，即%{address}%
            .withIgnorePaths("id","xxx")    // 忽略id和xxx字段，不管是什么值都不加入查询条件
            .withIgnoreCase()               // 忽略大小写
            .withIgnoreNullValues();        // 忽略空字段
    // 创建实例
    Example<Student> example = Example.of(student,matcher);
    return studentJpa.findAll(example);
}
```
之后在`StudentJPAController`类编写逻辑代码，打开浏览器访问即可。
```java
@RequestMapping(value = "/findByExample.do")
public List<Student> findByExample(Student student){
    return service.findByExample(student);
}
```
**ExampleMatcher.GenericPropertyMatcher方法**
+ caseSensitive(): 字符串区分大小写。
+ contains(): 全字符模糊匹配。
+ endsWith(): 结尾模糊匹配。
+ starsWith(): 开头模糊匹配
+ exact(): 精准匹配，也就是相等。
+ ignoreCase(): 字符串不区分大小写
+ storeDefaultMatching(): 默认匹配。
+ regex(): 正则表达式匹配


**StringMatcher参数**
<!-- Matching|生成语句|说明 -->
Matching|生成语句|说明
-----|:----: |---:
DEFAULT (case-sensitive)|firstname = ?0|默认（大小写敏感）
DEFAULT (case-insensitive)|LOWER(firstname) = LOWER(?0)|默认（忽略大小写）
EXACT (case-sensitive)|firstname = ?0|精确匹配（大小写敏感）
EXACT (case-insensitive)|LOWER(firstname) = LOWER(?0)|精确匹配（忽略大小写）
STARTING (case-sensitive)|firstname like ?0 + '%'|前缀匹配（大小写敏感）
STARTING (case-insensitive)|LOWER(firstname) like LOWER(?0) + '%'|前缀匹配（忽略大小写）
ENDING (case-sensitive)|firstname like '%' + ?0|后缀匹配（大小写敏感）
ENDING (case-insensitive)|LOWER(firstname) like '%' + LOWER(?0)|后缀匹配（忽略大小写）
CONTAINING (case-sensitive)|firstname like '%' + ?0 + '%'|模糊查询（大小写敏感）
CONTAINING (case-insensitive)|LOWER(firstname) like '%' + LOWER(?0) + '%'|模糊查询（忽略大小写）

补充：官方创建ExampleMatcher例子(1.8 lambda)
```java
ExampleMatcher matcher = ExampleMatcher.matching()
  .withMatcher("firstname", match -> match.endsWith())
  .withMatcher("firstname", match -> match.startsWith());
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除此之外，只要继承了`JpaRepository`接口，我们还能使用方法规则进行查询。举个例子，我在`StudentJpa`接口中定义一个`Student findByNameAndAge(String name,Integer age);`方法，那么它就可以直接被解析成：
```sql
select from stu where name=? and age=? 
```
是不是感觉很book思议？我第一次见也是感觉很神奇，一个简单的查询就这么在底层写个方法就被实现了，完全不用多写其他的东西，在这里提供了好多方法规则查询idea自带的方法提示，超级方便。只不过它的弊端就是对于复杂的操作语句，方法名会会变得很长，而且很难精准解析。
![规则查询](规则查询.png)

#### 3.3.5 自定义语句
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果想对SQL语句进行细致优化，我们还可以使用`@Query`注解自定义SQL语句。在`StudentJpa`接口中添加以下方法，并且自定义SQL语句。`nativeQuery`这个设置为true表明使用原生SQL，否则默认启用HQL。
```java
@Query(value = "select * from stu where age>=?",nativeQuery = true)
public List<Student> SelectByAge(Integer age);

@Transactional
@Modifying
@Query(value = "delete from stu where name=?",nativeQuery = true)
public void deleteByName(String name);
```
>
>在@Query 注解里设置value ，?1、?2 分别代表第一第二个参数。`@Query`只能用于查询，如果想用该注解实现其他操作类型就需要配合`@Modifying`注解一起使用，但是只是这么写的话会抛出一个`TranscationRequiredException`异常，意思就是当前操作需要开启事务，所以需要在这个前加上`@Transactional`注解开启自动化管理事务。
>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如同`@Query`注解一样，增删改查都可以使用原生SQL对数据库进行操作，所需要的注解分别是`@Insert`，`@Delete`，`@Update`，`@Select`。他们的在写SQL语句的时候取值可以使用`#{}`进行取值，内容下形参变量名。这几个属于**Mybatis**的注解，所以在使用的时候需要引入以下依赖。可以使用对应操作的注解，也可以使用`@Quey`加上另外两个注解配合使用。

```xml
<!-- mybatis依赖坐标 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
```

#### 3.3.6 自定义的BaseRepository

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;正常情况下一个项目肯定不可能就继承一个`JpaRepository`接口，再使用其他模块时还需要多个接口继承，如果每一个业务数据接口都继承几个相同的接口的话也不是不可以，但是对于系统设计和代码复用性来说不是个好的选择，这是我们可与创建一个我们自定的基础Repository。新建一个`base`包，在该包下创建一个`BaseRepositury`接口，并继承`JpaRepository`，日后使用其他模块时，在该接口进行添加即可。以后再创建Jpa接口只需要继承`BaseRepository`就行了，它有了`JpaRepository`所有实现方法。
```java
package com.example.demo.base;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.repository.NoRepositoryBean;
import java.io.Serializable;

/**
 * @Author: FBY
 * @Date: 2019/10/28 21:33
 * @Version 1.0
 */
@NoRepositoryBean
public interface BaseRepository<T,PK extends Serializable> extends JpaRepository<T,PK> {
}
```
>@NoRepositoryBean:这个注解如果配置在继承了JpaRepository接口以及其他SpringDataJpa内部的接口的子接口时，子接口不会 被作为一个Repository创建代理类。

#### 3.3.6 分页查询
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在一般的项目中，分页总是必不可少的，SpringDataJpa也内置了分页的方法。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先在`domian`包下创建一个`PageEntity`实体类，添加几个字段：当前页码、每页条数、排序列和排序方法。
```java
package com.example.demo.domain;

/**
 * @Author: FBY
 * @Date: 2019/10/28 21:37
 * @Version 1.0
 */
public class PageEntity {

    // 默认页码
    protected int page=1;
    // 默认每页数量
    protected int size=2;
    // 排序列名为id
    protected String sidx="id";
    // 排序规则
    protected String sord="desc";
    
    /*getter and setter*/
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;修改`Student`类继承`PageEntity`类，由于数据不多，这里测试就设定每页显示三条数据。在`StudentService`里面添加方法。
```java
//分页查询
List<Student> findAllPage(PageRequest pageRequest);
```
在`StudentServiceImpl`里添加实现。
```java
@Override
public List<Student> findAllPage(PageRequest pageRequest) {
    return studentJpa.findAll(pageRequest).getContent();
}
```
在`StudentJPAController`中添加新的方法，并添加对应的分页逻辑，此处分页的页码是从**0**开是的。
```java
@RequestMapping(value = "/page.do")
public List<Student> page(Integer page){
    Student student = new Student();
    student.setSize(3);
    student.setPage(page);
    return service.findAllPage(PageRequest.of(student.getPage()-1,student.getSize()));
}
```
接下来重启项目并访问该方法。
![测试分页查询](测试分页查询.png)

#### 3.3.7 排序
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`PageEntity`已经预设好了对应的排序字段，所以重新编辑page方法，将Sort对象添加在`PageRequest.of()`方法中就可以实现排序。我们现在将顺序按照id倒序排序，SpringDataJPA对排序方式添加了一个枚举类型，创建`Sort`对象时也需要枚举对象，因为我们`PageEntity`配置的是字符串，所以上面多了一步判断排序方法返回枚举对象。
```java
@RequestMapping(value = "/page.do")
public List<Student> page(Integer page){
    Student student = new Student();
    student.setSize(3);
    student.setPage(page);
    student.setSord("id");
    // 获取排序对象
    Sort.Direction sort_Direction = Sort.Direction.ASC.toString().equalsIgnoreCase(student.getSord()) ? Sort.Direction.ASC : Sort.Direction.DESC;
    // 设置排序对象
    Sort sort = new Sort(sort_Direction,student.getSidx());
    // 执行排序分页
    return service.findAllPage(PageRequest.of(student.getPage()-1,student.getSize(),sort));
}
```
重启项目，刷新页面即可。
<!-- ![测试排序分页](测试排序分页.png) -->
## 4. 总结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上就是SpringBoot整合SpringDataJPA的全部过程了，看完这篇，你就能简单使用JPA来实现项目需求了。是不是感觉很好用？对，它就是很好用。对于我们简单的数据处理真的很方便，很省事。但是大型项目中一些复杂的查询，比如一对多、多对多等，这些底层实现还是要自己动手写的，这些应该也有封装好的更方便的方法，至少对于写这篇笔记时候的我还不知道，以后慢慢了解慢慢学呗。就这么多吧，继续加油！！！