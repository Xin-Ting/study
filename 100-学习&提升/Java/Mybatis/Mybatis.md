
[技巧]![](img/Pasted%20image%2020211130091122.png)
Long、String等都实现了Serializable接口，写成Serializable接口类型会更通用。

# Mybatis-plus
## 1、解决了什么问题？
使用mybatis时，SQL语句比较繁琐，mybatis-plus简化了mybatis的开发，不用写SQL语句。
mybatis-plus提前写好了很多接口的方法，我们可以继承它的接口，但是我们需要告诉mybatis-plus我们要操作的表和条件。
## 2、什么是mybatis-plus？
一个基于mybatis框架提前定义好的一些通用的增删改查方法的工具包。
## 3、使用
- 引入依赖
```xml
<!--Springboot 集成 MybatisPlus依赖-->  
<dependency>  
 <groupId>com.baomidou</groupId>  
 <artifactId>mybatis-plus-boot-starter</artifactId>  
 <version>3.4.0</version>  
</dependency>  
<!-- mysql -->  
<dependency>  
 <groupId>mysql</groupId>  
 <artifactId>mysql-connector-java</artifactId>  
</dependency>  
<!-- 代码生成器 -->  
<dependency>  
 <groupId>com.baomidou</groupId>  
 <artifactId>mybatis-plus-generator</artifactId>  
 <version>3.4.0</version>  
</dependency>
<dependency>  
 <groupId>org.springframework.boot</groupId>  
 <artifactId>spring-boot-starter-freemarker</artifactId>  
</dependency>
```

- 配置文件
```properties
spring.datasource.driver-class-name=com.mysql.jdbc.Driver  
spring.datasource.url=jdbc:mysql:///mybatis?characterEncoding=utf8&serverTimezone=Asia/Shanghai  
spring.datasource.username=root  
spring.datasource.password=root

```

- 继承BaseMapper接口
![](img/Pasted%20image%2020211130094443.png)
- 指定表名
![](img/Pasted%20image%2020211130094546.png)
[技巧]![](img/Pasted%20image%2020211130095349.png)

## 4、 常用注解
1. ``@TableName(“tb_user”)``：当实体类名和表名不一致的时候，可以通过这个注解来做对应关系；
	当在配置文件中配置表的前缀，则可以不用写这个注解。
	![](img/Pasted%20image%2020211130143423.png)
2. ``@TableId(type=IdType.AUTO)``：指定主键生成策略，mybatis-plus在进行数据插入的时候需要知道主键怎么赋值：
	- 自增长
	- UUID
	- 雪花算法
	可在配置文件中全局配置
	![](img/Pasted%20image%2020211130143449.png)
3. ``@TableField(“user_name”)``：当实体中的属性名称和表中的字段名称不一致时，可以通过这个注解来做对应关系；
	- ![](img/Pasted%20image%2020211130101447.png)
	- 注意：![](img/Pasted%20image%2020211130101815.png)
4. ``@TableField(exist=false)``:当实体中的属性不在表中的时候，可以排除这个属性，通常用于多表关联。 
	- ![](img/Pasted%20image%2020211130101129.png)


## 5、条件构造器
重点掌握``LambdaQueryWrapper<>()``，可以查询、修改、删除,也可以用``Wrappers.lambdaQuery()``。
![](img/Pasted%20image%2020211130103129.png)

- 模糊查询
	 ![](img/Pasted%20image%2020211130110550.png)
- 与查询
	![](img/Pasted%20image%2020211130110817.png)
- 或查询
	![](img/Pasted%20image%2020211130111020.png)
- 分页插件(需要分页插件）
```java

// MP 分页插件  
@Bean  
public MybatisPlusInterceptor mpInterceptor() {  
    PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();  
 MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();  
 mybatisPlusInterceptor.addInnerInterceptor(paginationInnerInterceptor);  
 return mybatisPlusInterceptor;  
}
```


## 6、为什么要用Wrappers来创建QueryWrapper？
使用案例：

```java
LambdaQueryWrapper<Demo> lambdaQueryWrapper = Wrappers.lambdaQuery();
lambdaQueryWrapper.eq(Demo::getName, name);
```

Wrappers
```java
public final class Wrappers {
    private Wrappers() {
        // ignore
    }

    /**
     * 获取 LambdaQueryWrapper< T >
     *
     * @param <T> 实体类泛型
     * @return LambdaQueryWrapper< T >
     */
    public static <T> LambdaQueryWrapper<T> lambdaQuery() {
        return new LambdaQueryWrapper<>();
    }
    // 省略一些其他方法定义
}
```

**核心原理**：使用==静态工厂方法模式==
说明：屏蔽了创建对象的细节，比如，创建对象之前他需要做一个初始化动作，你的每个new对象都要改，如果调用方法就不用你改动，方法的实现改动一下就可以了。

注意其中的构造方法被声明为了 private，这样可以防止它被外部调用，于是调用方在使用实例的时候，基本上就必须通过 lambdaQuery、lambdaQuery、emptyWrapper 这几个静态工厂方法来创建。
对外暴露的属性越多，调用者就越容易出错。
所以对于类的提供者，一般来说，应该努力减少对外暴露属性，从而降低调用者出错的机会。
能够扩大类的提供者对自己所提供的类的控制力。
**缺点**
缺类的所有重载的构造方法的名字都相同，不能从名字上区分每个重载方法，容易引起混淆。
静态工厂方法的方法名可以是任意的，这一特性的优点是可以提高程序代码的可读性

## 7、多表
 **mybatis-plus只能操作单表的CRUD，多表需要自己写，与mybatis-plus无关**
 
 参照之前的mybatis多表查询。
 
 mybatis中表的关系有几种？
 两种：一对一，一对多。(多对多可以拆分成一对多)
 
 多表联查不要用内连接，不能延迟加载，==用映射文件查询。==(掌握)
 -  配置文件：
     classpath：仅仅扫描自己写的代码的resources目录
	 classpath*：除了自己写的resources目录下还包括所有的子依赖包中的resources目录下的mapper文件
	![](img/Pasted%20image%2020211130152047.png)