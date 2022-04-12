---
title: Mybatis小知识点
route: mybatis-knowPoints
date: 2019-9-3 23:32:32
tags: [Mybatis,Java]
categories: Mybatis
image: /images/cover/mybatis-knowPoints.png
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在这简单介绍了一下mybatis里的几个小知识点，分别是`parameterMap`与`parameterTyppe`、`resultMap`与`resultType`的区别，以及在xml文件中获取传递过来的值。同时讲解了一下怎么自定义多参数进行传值等。
<!-- more -->
# Mybatis小知识点
## 1. parameterMap与parameterTyppe详解
### 1.1 parameterMap类型
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`parameterMap`和`resuletMap`类似，表示将查询结果集中列值的类型一一映射到Java对象属性的类型，在开发过程中不推荐这种方式。
### 1.2 parameterType类型
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`parameterType`声明输入参数的类型，将会传入这条语句中的参数类的完全限定名或别名，这个属性是可选的，因为Mybatis可以通过类型处理器`(TypeHandler)`推断出具体传入语句的参数，默认值为未设置`(unset)`。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果SQL语句接收的参数是一个JavaBean对象，要在里面写该JavaBean对象的类的全名，即`包名.类名`，如果再开头mapper中已经配置了`namespace`别名，那么只要接收参数是这个mapper包下的，就可以直接写文件名。如果接受的是基本类型，那么久可以直接写基本类型，因为基本类型在这里已经自动被扫描了，所以不需要再写包名。如果传参的参数是集合，那么只需要写集合中的对象的类型即可,而不是集合本身。

## 2. resultMap与resultType详解
### 2.1 resultMap类型
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;外部`resultMap`的命名引用。表示将查询结果集中的列一一映射到bean对象的各个属性。映射的查询结果集中的列标签可以根据需要灵活变化，并且，在映射关系中，还可以通过typeHandler设置实现查询结果值的类型转换，比如布尔型与0/1的类型转换。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在外部可以自定义一个`resultMap`。注入返回值类型 ,自定义结果集映射规则，自定义某个JavaBean的封装规则.如以下代码：
```xml
<!-- id：唯一id，方便引用。
    type：自定义规则的Java类。 -->
<resultMap type="Orders" id="orders">
    <!-- result：定义普通列封装规则 
            column：指定数据库中的哪一列列名
            property：指定对应的javaBean中的属性名
            jdbcType：指定对应数据库中属性的类型
    -->
    <result column="User_id" property="userId" jdbcType="INTEGER"/>
    <result column="User_name" property="userName"  jdbcType="VARCHAR"/>
    <!--其他不指定的列会自动封装：我们只要写resultMap，就尽量把所有的列都写上 -->
    <!-- 在写SQL的返回值时在resultMap="orders",你想返回哪个自定义的mapper，你就把自定义哪个的id写上。-->
</resultMap>
```

### 2.2 resultType类型
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用resultType时我们应该注意:sql查询的列名要和resultType指定pojo的属性名相同，指定相同属性方可映射成功，如果sql查询的列名要和resultType指定pojo的属性名全部不相同(或是部分不相同)，则映射到pojo对象中的对应属性为null。例如有时候我们不需要查询`select * from user where id = ?`而是`select username,address _address where id = ?` 此时我们给查询的`address`列名给了一个别名`_address`，这样我们通过查询表中`address`的数据然后在将它映射到User对象时，该对象的`address`属性就为`null`，即没将从表中查询到的`address`数据映射到user对象的`address`属性中。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从这条语句中返回的期望类型的类的完全限定名或别名。 注意如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身。可以使用 `resultType` 或 `resultMap`，但不能同时使用。

## 3. \#{}与\${}详解
### 3.1 \#{}取值
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`#{}`输入参数的占位符，#将传入的数据都当成一个字符串，会对传入的数据自动加上引号。在取值的时候mybatis会进行预编译。如果入参只有一个值，那么大括号中写什么多无所谓，如果传参有多个，还是写入参属性名写。最好规范一点都按入参属性名写，这样能一眼看出入参是什么。示例(ID=6)如下：
```sql
select * from user where id=#{ID}
```
会被先编译成
```sql
select * from user where id=?
```
然后再用ID的值(6)去替代`?`。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`\#{}`因为有预编译，显得更安全。如果传入参数值中有`#`使用`#{}`,*(`#`在sql中表示注释)*，不会使`#`后面的sql失效。当入参的name='hh#'，示例如下：
用`\#{}`:
```sql
select * from user where name=#{name} and id=#{ID}
```
执行的sql为
```sql
select * from user where name='hh#' and id=6
```
用`\${}`:
```sql
select * from user where name=${name} and id=${ID}
```
执行的sql为
```sql
select * from user where name='hh' # and id=6
```
因为`#`表示注释，所以执行效果相当于
```sql
select * from user where name='hh'
```

**`#{}`可以指定其他属性**
        如果name传入参数的值为null，mybatis会默认name值为other类型，但是oracle数据库不能处理other类型，因为会报不能识别的错误。此使就可以用jdbcType属性指定类型。
```sql
select * from user where name=#{name,jdbcType=null} and id=${ID}
```
当然也可以在mybatis配置文件中进行配置。
```xml
<settings>
   <setting name="jdbcTypeForNull" value="NULL">
</settings>
```

### 3.2 ${}取值
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`${}`使直接进行编译，将传入的数据直接显示生成在SQL中，使用它可能会导致SQL注入攻击，能用`#{}`的地方就不用`${}`，但是写`order by`句子的时候一定用`${}`。如果入参只有一个值，那么大括号中要写`value`，如果传参有多个，还是写入参属性名写。示例如下(ID=6)
```sql
select * from user id=${ID}
```
执行的sql为
```sql
select * from user id=6
```
用`${}`模糊查询时(name=张)
```sql
select * from user where username like '%${value}%' 
```
字符串拼接之后编译成
```sql
select * from user where username like '%张%'
```

jdbc不支持占位符的地方可以用`${}`进行取值，比如表名和排序字段：
```sql
select * from ${user} where name=#{name} and id=#{ID} order by ${name}
```

## 4. sql语句接收多个参数
### 4.1 JavaBean方法传值
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当要向后台sql语句传入的参数个数是多个时，可以将要传入的参数封装成一个JavaBean，然后将这个JavaBean对象传入到mapper接口，在xml文件的sql语句通过属性名来进行取值。

### 4.2 Map方法传值
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;向后台sql语句传递参数多于一个，又不方便封装成JavaBean是，可以将参数封装成Map集合，以键值对`(key-value)`的形式存储，然后将该Map集合传入后台xml文件。在xml文件中，执行该语句的接收参数属性设置为`parameterType="Map"`，然后在sql中取值的属性名要和map中的键值对的键`(key)`一致。
```java
private Mapper mapper；
Map map = new HashMap();
map.put("id",1);
map.put("name","张三");
//调用mapper接口
mapper.insert(map);
```
下面是mapper中的方法
```java
int insert(Map map);
```
下面是xml中的方法
```xml
<insert id="insert"parameterType="Map">
    insert into user (id,name) values(#{id},#{name})  
</insert>
```

### 4.3 @Param方法传值

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当参数较少时，也可以通过在接口的方法里注解`@Prama`，可以同时注解多个参数。
**创建接口方法**
```java
/**
 * 根据姓名和性别模糊查询
 * @Param username 用户姓名
 * @Param sex 用户性别
 * @return User对象集合
 */
List<User> select(@Param("name")String username,@Param("sex")String sex);
```
@Param("name")就是告诉mybatis，参数username在SQL语句中用name作为key，也就是说，mybatis帮我们完成了调用时，类似param.put("name",username);

**配置SQl语句**
```xml
<select id="select" resultMap="BaseResultMap">
    select * from user where name like '%${name}%' and sex=#{sex}
</select>
```
此处的`#{name}`对应的是@Param("name")中的name，需要完全一致。

**调用**
最后在调用时只需要按参数提示直接传入对应的实际参数即可。

## 5. 打印SQL语句
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在项目测试阶段，有时候我们需要查看调用的SQL语句是那一句，可以通过log4j进行输出SQL语句，mybatis只需要配置一下配置文件就行了，在mybatis.xml文件中添加以下两行代码就可以，这样在调用到那句SQL语句的时候就可以对该语句进行输出在控制台，以方便查看。
```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true" />
    <setting name="logImpl" value="STDOUT_LOGGING" />
</settings>
```
