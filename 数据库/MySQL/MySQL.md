# 1、什么是MySQL？
**介绍**：
		MySQL是一个**关系型数据库**管理系统的一种。由瑞典MySQL AB公司开发，后来被Oracle公司收购。**是一个用于存储和管理数据的仓库。**
		MySQL 的默认端口号是**3306**
**关系型数据库：** 是指将数据保存在不同的数据表中，而不是将所有的数据放在一个大仓库内，而且表与表之间可以有关联关系(一对一、一对多、多对多)，这样提高了访问速度和灵活性。
		![关系型数据库](img/Pasted%20image%2020211220160239.png)
常见的关系型数据库：MySQL、PostgreSQL、Oracle、SQL Server、SQLite（微信本地的聊天记录的存储就是用的 SQLite）等。
# 2、解决什么问题？
1. ==可以持久化存储数据==
2. ==方便存储和管理数据==
3. 使用了统一的方式操作数据库

# 3、数据库的组成及关系
1. 数据库
	- 用于存储和管理数据的仓库
	- 一个库中可以包含多个数据表
2. 数据表
	- 数据库最重要的组成部分之一
	- 由纵向的列和横向的行组成
	- 可以指定列名、数据类型、约束等
	- 一个表中可以存储多条数据
3. 数据
	- 想要永久化存储的数据

>每一种数据库操作的方式可能会存在一些不一样的地方，我们称之为`方言`
>MySQL不区分大小写

#think **mysql有关权限的表都有哪几个**

MySQL服务器通过权限表来控制用户对数据库的访问，权限表存放在mysql数据库里，由mysql_install_db脚本初始化。这些权限表分别user，db，table_priv，columns_priv和host。下面分别介绍一下这些表的结构和内容：
1. user权限表：记录允许连接到服务器的用户帐号信息，里面的权限是全局级的。
2. db权限表：记录各个帐号在各个数据库上的操作权限。
3. table_priv权限表：记录数据表级的操作权限。
4. columns_priv权限表：记录数据列级的操作权限。
5. host权限表：配合db权限表对给定主机上数据库级操作权限作更细致的控制。这个权限表不受GRANT和REVOKE语句的影响。

#think **MySQL的binlog有有几种录入格式？分别有什么区别？**

有三种格式，statement，row和mixed。

1. statement模式下，每一条会修改数据的sql都会记录在binlog中。不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。由于sql的执行是有上下文的，因此在保存的时候需要保存相关的信息，同时还有一些使用了函数之类的语句无法被记录复制。
2. row级别下，不记录sql语句上下文相关信息，仅保存哪条记录被修改。记录单元为每一行的改动，基本是可以全部记下来但是由于很多操作，会导致大量行的改动(比如alter table)，因此这种模式的文件保存的信息太多，日志量太大。
3. mixed，一种折中的方案，普通操作使用statement记录，当无法使用statement的时候使用row。

此外，新版的MySQL中对row级别也做了一些优化，当表结构发生变化的时候，会记录语句而不是逐行记录。

# 4、SQL分类
1. DDL(Data Definition Language)数据定义语言
	- 用来定义数据库对象：数据库，表，列等。关键字：create, drop,alter 等
2. **DML(Data Manipulation Language)数据操作语言**
	- 用来对数据库中表的数据进行增删改。关键字：insert, delete, update 等
3. **DQL(Data Query Language)数据查询语言**
	- 用来查询数据库中表的记录(数据)。关键字：select, where 等
4. DCL(Data Control Language)数据控制语言(了解)
	- 用来定义数据库的访问权限和安全级别，及创建用户。关键字：GRANT， REVOKE 等

# 5、MySQL使用
## 5.1 DDL操作数据库和数据表

**创建数据库：**

```mysql
-- 标准语法
CREATE DATABASE 数据库名称;

-- 创建db1数据库
CREATE DATABASE db1;

-- 创建一个已存在的数据库会报错
-- 错误代码：1007  Can't create database 'db1'; database exists
CREATE DATABASE db1;

-- 标准语法
CREATE DATABASE IF NOT EXISTS 数据库名称;

-- 创建数据库db2(判断，如果不存在则创建)
CREATE DATABASE IF NOT EXISTS db2;
```

 ==**创建数据表**==

```mysql
-- 创建表
	create table 表名(
	    列名1 数据类型 [约束],
	    列名2 数据类型 [约束],
	    ...
	);
	
  常用数据类型:
	    数值型
	        int
	        double(总长度,小数位数)
	    
	    字符型
		varchar(长度值)  -- 可变长度；长度值指的是最大长度
			如果数据的长度不确定，选择使用这个
		char(长度值)  -- 固定长度；长度值指的是这个数据必须占据的长度大小
			如果数据的长度是确定的，选择用这个
	    
	    时间型
		date: yyyy-MM-dd
		datetime:yyyy-MM-dd HH:mm:ss，如果没有给值，会使用null作为值
		timestamp:yyyy-MM-dd HH:mm:ss,时间戳，如果没有给值，会使用当前系统时间作为值
		
-- 增加列
	alter table 表名 add 列名 数据类型 [约束];
```

#think **mysql有哪些数据类型**

1. 整数类型，包括TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT，分别表示1字节、2字节、3字节、4字节、8字节整数。
	- 任何整数类型都可以加上UNSIGNED属性，表示数据是无符号的，即非负整数。  
	- 长度：整数类型可以被指定长度，例如：INT(11)表示长度为11的INT类型。长度在大多数场景是没有意义的，它不会限制值的合法范围，只会影响显示字符的个数，而且需要和UNSIGNED ZEROFILL属性配合使用才有意义。  
	- 例子，假定类型设定为INT(5)，属性为UNSIGNED ZEROFILL，如果用户插入的数据为12的话，那么数据库实际存储数据为00012。

2. 实数类型，包括FLOAT、DOUBLE、DECIMAL。  
	- DECIMAL可以用于存储比BIGINT还大的整型，能存储精确的小数。  
	- 而FLOAT和DOUBLE是有取值范围的，并支持使用标准的浮点进行近似计算。  
	- 计算时FLOAT和DOUBLE相比DECIMAL效率更高一些，DECIMAL可以理解成是用字符串进行处理。

3. 字符串类型，包括VARCHAR、CHAR、TEXT、BLOB 。
	- VARCHAR用于存储可变长字符串，它比定长类型更节省空间。VARCHAR使用额外1或2个字节存储字符串长度。列长度小于255字节时，使用1字节表示，否则使用2字节表示。VARCHAR存储的内容超出设置的长度时，内容会被截断。  
	- CHAR是定长的，根据定义的字符串长度分配足够的空间。CHAR会根据需要使用空格进行填充方便比较。CHAR适合存储很短的字符串，或者所有值都接近同一个长度。CHAR存储的内容超出设置的长度时，内容同样会被截断。  
	- **使用策略：**  
		- 对于经常变更的数据来说，CHAR比VARCHAR更好，因为CHAR不容易产生碎片。  
		- 对于非常短的列，CHAR比VARCHAR在存储空间上更有效率。  
		- 使用时要注意只分配需要的空间，更长的列排序时会消耗更多内存。  
		- 尽量避免使用TEXT/BLOB类型，查询时会使用临时表，导致严重的性能开销。

4. 枚举类型（ENUM），把不重复的数据存储为一个预定义的集合。  
	- 有时可以使用ENUM代替常用的字符串类型。ENUM存储非常紧凑，会把列表值压缩到一个或两个字节。ENUM在内部存储时，其实存的是整数。 尽量避免使用数字作为ENUM枚举的常量，因为容易混乱。排序是按照内部存储的整数。

5. 日期和时间类型，尽量使用timestamp，空间效率高于datetime，用整数保存时间戳通常不方便处理。如果需要存储微秒，可以使用bigint存储。  

## 5.2 DML操作

==注意：值的类型为字符型/日期型，值必须用引号引起来==

```mysql
DML:
	****值的类型为字符型/日期型，值必须用引号引起来
	
	-- 指定列插入
	insert into 表名 (列名1,列名2,...) values (值1,值2,...);
	-- 全列插入：值的个数和类型要和表中列数和类型一致
	insert into 表名 values (值1,值2,...);  
	
	-- 修改记录（执行顺序：update-->where-->set）
	update 表名 set 列名1=值1,列名2=值2,... [where 条件]
	
	-- 删除记录（执行顺序：from-->where-->delete）
	delete from 表名 [where 条件]
```

> 注意：修改和删除一定要加where条件，不然就会修改或者删除整个列或者整个表；
> 列名和值的数量以及数据类型要对应。

## 5.3 DQL操作

 ==**编写顺序：**==

```mysql
select
	字段列表
from
	表名列表
where
	条件列表  
group by
	分组字段
having
	分组之后的条件
order by
	排序         -- 注意：多个排序条件，当前边的条件值一样时，才会判断第二条件
limit
	分页限定		
-- 分页查询
-- limit begin,pageSize  begin指从哪个索引开始查，索引号是从0开始的
-- begin = (当前页的页码-1)*pageSize
-- pageSize 是每页多少个

-- where和having的区别是什么？
-- where是分组前执行，having是分组后执行，having后可以使用聚合函数和别名，但是where不可以
```

 ==**查询顺序：**==

```mysql
from...
where...
group by...
having...
select...
order by... 
limit...
```

![查询顺序1](img/Pasted%20image%2020211220144523.png)
![查询顺序2](img/Pasted%20image%2020211220144558.png)
**条件查询**
- 条件分类

| 符号                    | 功能                                       |
| ----------------------- | ------------------------------------------ |
| >                       |                                            |
| <                       | 小于                                       |
| >=                      | 大于等于                                   |
| <=                      | 小于等于                                   |
| =                       | 等于                                       |
| <> 或 !=                | 不等于                                     |
| ==BETWEEN ... AND ...== | ==在某个范围之内(都包含)==                 |
| ==IN(...)==             | ==多选一==                                 |
| ==LIKE 占位符==         | ==模糊查询  _单个任意字符  %多个任意字符== |
| IS NULL                 | 是NULL                                     |
| IS NOT NULL             | 不是NULL                                   |
| AND 或 &&               | 并且                                       |
| OR 或 \|\|              | 或者                                       |
| NOT 或 !                | 非，不是                                   |

**聚合函数**
 - 将一列数据作为一个整体，进行纵向的计算
 - 聚合函数分类

| 函数名      | 功能                           |
| ----------- | ------------------------------ |
| count(列名) | 统计数量(一般选用不为null的列) |
| max(列名)   | 最大值                         |
| min(列名)   | 最小值                         |
| sum(列名)   | 求和                           |
| avg(列名)   | 平均值                         |

> count(*)/count(1) 是以行为统计单位，只要这一行存在，不管行中的列的值是什么，都会记为1

# 6、约束	

## 6.1   单表

1. **主键约束：primary key，该列的值是唯一且非空的**。一个表中只能有一个主键，但是这个主键可以由多个列共同构成（联合主键）。
	- 主键在开发中的意义是用来标识一条记录唯一性的，至于主键的值是多少，我们并不关心，所以开发中int类型的主键，会结合auto_increment一起使用，利用myslq的自动生成策略去生成主键的值。

2. **唯一约束: unique ,该列的值不允许重复，可以是NULL，可以有多个NULL。**
> 可以输入多个字符串null，MySQL5.7版本之后会在数据库中会转换为NULL 。

3. **非空约束: not null,该列的值不允许为NULL**

> **唯一约束可以和非空约束一起使用，这样就唯一且非空。**

**语法：**

```mysql
-- 添加约束语法：
	****建表时添加：
	create table 表名(
		列名 数据类型 约束,
		...
	);
	
	-- 修改表添加
	alter table 表名 modify 列名 数据类型 约束;

-- 删除约束语法：
	主键：alter table 表名 drop primary key;
	唯一：alter table 表名 drop index 列名;
	非空: alter table 表名 modify 列名 数据类型;
-- ============================================================
-- 主键约束
CREATE TABLE test1(
	id INT PRIMARY KEY,
	NAME VARCHAR(20)
);
DESC test1;   -- 查询test1表
ALTER TABLE test1 DROP PRIMARY KEY;

CREATE TABLE test2(
	id INT,
	NAME VARCHAR(20),
	age INT,
	PRIMARY KEY (id,NAME)   -- 联合主键
);

CREATE TABLE test3(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20)
);

-- 唯一约束
CREATE TABLE test4(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20) UNIQUE
);

-- 非空约束
CREATE TABLE test5(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20) UNIQUE NOT NULL,
	tel VARCHAR(11) UNIQUE NOT NULL
);
```

## 6.2  多表

**外键约束：**  foreign key,描述关系型数据库中多表间关系的一种机制，通过这种机制可以保证多个有关系的表中的数据的有效性和完整性。

作用：让表和表之间产生关系，从而保证数据的准确性！

语法：

```mysql
-- 外键约束创建 （语法了解）
	主表(
		id int primary key auto_increment,
		....
	);
	从表(
		id int primary key auto_increment,
		...,
		外键列名 int,
		constraint fk_从表名_主表名 foreign key (外键列名) references 主表名(id)
	);
	-- 外键约束删除(语法了解)
	alter table 从表名 drop foreign key 外键名;
```

#think  **SQL的约束有哪几种？**

![数据库约束](img/Pasted%20image%2020211220192657.png)
对于数据库来说，约束分为列约束（Column Constraint）和表约束（Table Constraint）。  
列约束作为列定义的一部分只作用于此列本身。表约束作为表定义的一部分，可以作用于多个列。
其中列约束共有六种，分别是：主键约束、外键约束、唯一约束、检查约束、默认约束和非空约束；
表约束共有四种，分别是：主键约束、外键约束、唯一约束和检查约束。
1. **主键约束：primary key，该列的值是唯一且非空的**。一个表中只能有一个主键，但是这个主键可以由多个列共同构成（联合主键）。
	- 主键在开发中的意义是用来标识一条记录唯一性的，至于主键的值是多少，我们并不关心，所以开发中int类型的主键，会结合auto_increment一起使用，利用myslq的自动生成策略去生成主键的值。
2. **唯一约束: unique ,该列的值不允许重复， 一个表允许有多个 Unique 约束，可以是NULL，可以有多个NULL。**
3.  **非空约束: not null,该列的值不允许为NULL,** 唯一约束可以和非空约束一起使用，这样就唯一且非空。
4.  **外键约束：  foreign key,** 描述关系型数据库中多表间关系的一种机制，通过这种机制可以保证多个有关系的表中的数据的有效性和完整性。让表和表之间产生关系，从而保证数据的准确性！
5.  **检查约束：check，约束用于限制字段中的值的范围。**
6.  **默认约束： default ，默认值** 如果定义了默认值，再插入数据时如果没有插入数据，会根据默认值插入
# 7、多表设计

##  ==7.1多表间的关系==
 1. **一对一**
 	- 唯一外键：在任意一方创建一个外键字段，关联另一方的主键列，这个外键列需要添加唯一约束
 	- 主键对应：将两个表的主键值一一对应
2. **一对多**
	- 在多方表中添加一个列，作为外键列去关联一方表的主键
3. **多对多**
	- 创建一张三方表，最少包含两个字段，分别作为外键关联两张多方表的主键

##  7.2多表查询

### 7.2.1 笛卡尔积

- 笛卡尔积查询:获取这两个表的所有组合情况

```mysql
-- 标准语法
SELECT 列名 FROM 表名1,表名2,...;

-- 查询user表和orderlist表
SELECT * FROM USER,orderlist;
```

### ==7.2.2内连接查询==

内连接查询的是两张表有交集的部分数据(有主外键关联的数据)	   

```mysql
**隐式内连接（sql 1992）
	select * from 表1,表2,... where 主外键对等;
	    
显示内连接(sql 1995)
    select * from 表1 [inner] join 表2 on 主外键对等 where 其他条件;
```

### ==7.2.3外连接查询==

外连接：两个表的交集数据+主表全部

```mysql
**左外连接
	select * from 表1 left [outer] join 表2 on 主外键对等 where 其他条件;  -- 表1为主表
右外连接
    select * from 表1 right [outer] join 表2 on 主外键对等 where 其他条件;	-- 表2为主表
```

### ==7.2.4子查询==

子查询: 一条查询语句中嵌套另外一条查询语句，被嵌套的查询语句称之为子语句
**子语句的查询结果：**

 **1.单行单列的（单值）**

- 可以放在where后面，跟单值运算符
- 可以放在select后面
- 可以放在having后面，跟单值运算符

```mysql
-- 标准语法
SELECT 列名 FROM 表名 WHERE 列名=(SELECT 聚合函数(列名) FROM 表名 [WHERE 条件]);

-- 查询年龄最高的用户姓名
SELECT MAX(age) FROM USER;              -- 查询出最高年龄
SELECT NAME,age FROM USER WHERE age=26; -- 根据查询出来的最高年龄，查询姓名和年龄
SELECT NAME,age FROM USER WHERE age = (SELECT MAX(age) FROM USER);
```

 **2.多行单列(一列有多个值)**

- 可以放在where后面，跟in运算符

```mysql
-- 标准语法
SELECT 列名 FROM 表名 WHERE 列名 [NOT] IN (SELECT 列名 FROM 表名 [WHERE 条件]); 

-- 查询张三和李四的订单信息
SELECT id FROM USER WHERE NAME='张三' OR NAME='李四';   -- 查询张三和李四用户的id
SELECT number,uid FROM orderlist WHERE uid=1 OR uid=2; -- 根据id查询订单
SELECT number,uid FROM orderlist WHERE uid IN (SELECT id FROM USER WHERE NAME='张三' OR NAME='李四');
```

**3.多行多列/单行多列（表结构）**

- 可以放在from后面，当成一张虚拟表使用

```mysql
-- 标准语法
SELECT 列名 FROM 表名 [别名],(SELECT 列名 FROM 表名 [WHERE 条件]) [别名] [WHERE 条件];

-- 查询订单表中id大于4的订单信息和所属用户信息
SELECT * FROM USER u,(SELECT * FROM orderlist WHERE id>4) o WHERE u.id=o.uid;
```

**==如果需求中有聚合操作，基本都会用到子查询==**    

# 8、视图

视图：对一条查询语句的封装，占据大小只是一条查询语句的大小。
	 1. 可以简化复杂查询语句的复用
	 2. 屏蔽一些敏感的列的数据，可以通过视图查询某些指定的列

```mysql
 	创建：
	create view 视图名 [(列1,列2,...)] as 查询语句;
	
    使用:
	select * from 视图名;
	
    删除：
	drop view [if exists] 视图名;
```

#think **为什么要使用视图？什么是视图？**

- 为了提高复杂SQL语句的复用性和表操作的安全性，MySQL数据库管理系统提供了视图特性。所谓视图，本质上是一种虚拟表，在物理上是不存在的，其内容与真实的表相似，包含一系列带有名称的列和行数据。但是，视图并不在数据库中以储存的数据值形式存在。行和列数据来自定义视图的查询所引用基本表，并且在具体引用视图时动态生成。

- 视图使开发者只关心感兴趣的某些特定数据和所负责的特定任务，只能看到视图中所定义的数据，而不是视图所引用表中的数据，从而提高了数据库中数据的安全性。

#think **视图有哪些特点？**

视图的特点如下:
- 视图的列可以来自不同的表，是表的抽象和在逻辑意义上建立的新关系。
- 视图是由基本表(实表)产生的表(虚表)。
- 视图的建立和删除不影响基本表。
- 对视图内容的更新(添加，删除和修改)直接影响基本表。
- 当视图来自多个基本表时，不允许添加和删除数据。

#think **视图的使用场景有哪些？**

视图根本用途：简化sql查询，提高开发效率。如果说还有另外一个用途那就是兼容老的表结构。

下面是视图的常见使用场景：
- 重用SQL语句；
- 简化复杂的SQL操作。在编写查询后，可以方便的重用它而不必知道它的基本查询细节；
- 使用表的组成部分而不是整个表；
- 保护数据。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限；
- 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据。

#think  **视图的优点**

1. 查询简单化。视图能简化用户的操作
2. 数据安全性。视图使用户能以多种角度看待同一数据，能够对机密数据提供安全保护
3. 逻辑数据独立性。视图对重构数据库提供了一定程度的逻辑独立性

#think **视图的缺点**

1. 性能。数据库必须把视图的查询转化成对基本表的查询，如果这个视图是由一个复杂的多表查询所定义，那么，即使是视图的一个简单查询，数据库也把它变成一个复杂的结合体，需要花费一定的时间。
2. 修改限制。当用户试图修改视图的某些行时，数据库必须把它转化为对基本表的某些行的修改。事实上，当从视图中插入或者删除时，情况也是这样。对于简单视图来说，这是很方便的，但是，对于比较复杂的视图，可能是不可修改的。这些视图有如下特征：1.有UNIQUE等集合操作符的视图。2.有GROUP BY子句的视图。3.有诸如AVG\SUM\MAX等聚合函数的视图。 4.使用DISTINCT关键字的视图。5.连接表的视图（其中有些例外）

#think **什么是游标？**

游标是系统为用户开设的一个数据缓冲区，存放SQL语句的执行结果，每个游标区都有一个名字。用户可以通过游标逐一获取记录并赋给主变量，交由主语言进一步处理。
# 9、存储过程

存储过程：相当于是java中没有返回值的方法,封装sql业务
     1. 方便sql业务的复用
     2. sql业务是事先编译好存储在数据库服务器端的，后续我们在执行操作时，就不需要在编译了，直接执行，相对而言效率更高
    
```mysql
创建：
-- 修改默认结束符
    delimiter $   -- 中间要有一个空格
    create procedure 存储过程名(参数列表)
    begin
        sql业务;
    end$
    -- 将结束符修改回;
    delimiter ;
	
调用：
    call 存储过程名(实际参数);
    
参数列表：
构成：参数类型 参数名 数据类型
参数类型：
     in: 默认值，入参
     out: 出参，将sql业务的结果传递到存储函数外使用
     inout: 出入参，既可以接收用户传入的值，又可以将业务的结果传递出去
    
sql业务：
     变量
     定义
         declare 变量名 数据类型 [default 默认值];
     
     赋值
         set 变量名 = 值;
         select 列1,列2,... into 变量1,变量2,... from 表名 where 条件; -- 这条查询语句只能是一行记录
     
     使用
         select 变量名1,变量名2,...
         
     -- ***注意：
	-- ① sql业务中所有变量的定义必须放在最开始的位置
	-- ② 数值类型的变量要参与运算，必须有初始值，否则初始值为NULL
     
     if判断
         if 条件判断1 then
              sql业务1;
         [elseif 条件判断2 then
	  sql业务2;
         ]
         [else
              sql业务;
         ]
         end if;
     
     while循环
     while 条件判断 do
         sql业务;
     
         条件控制;
     end while;
```

```mysql
-- 存储过程语法
-- 创建
DELIMITER $
CREATE PROCEDURE pro_test1()
BEGIN
	SELECT * FROM student;
END$
DELIMITER ;
-- 调用
CALL pro_test1();

-- =====================================================================================

-- 演示变量
DELIMITER $
CREATE PROCEDURE pro_test2()
BEGIN 
     -- 定义变量
     DECLARE v1 INT DEFAULT 0; -- 演示运算 
     DECLARE v2 VARCHAR(20); -- 演示字符数据赋值
     DECLARE v3 DOUBLE(4,2); -- 记录平均年龄
     DECLARE v4 INT; -- 记录总人数
     -- 赋值方式一
     SET v1 = v1 + 1;
     SET v2 = '统计';
     -- 赋值方式二
     SELECT AVG(age),COUNT(1) INTO v3,v4 FROM student;
     
     -- 使用变量
     SELECT v2,v1,v3,v4;
     
END$
DELIMITER ;
CALL pro_test2();
-- =====================================================================================

-- 需求一：演示if，随机生成一个100以内的整数，如果是奇数： 输出  xx 是 奇数  如果是偶数 输出 xx 是 偶数
SELECT RAND()*100; -- 生成随机数
SELECT CEIL(1.2);  -- 向上取整
SELECT FLOOR(1.2); -- 向下取整
SELECT CEIL(RAND()*100); 

SELECT CONCAT('1','是一个偶数'); -- 拼接字符串

DELIMITER $
CREATE PROCEDURE pro_test3()
BEGIN
	DECLARE num INT; -- 记录随机数
	SET num = CEIL(RAND()*100); -- 赋值
	
	-- 判断奇偶数
	IF num%2=0 THEN
		SELECT CONCAT(num,'是一个偶数');
	ELSE
		SELECT CONCAT(num,'是一个奇数');
	END IF;
END$
DELIMITER ;
CALL pro_test3();
-- ====================================================================================

/*
需求二：演示参数类型，结果通过出参的方式传递
	通过学生的id，查询学生分数
	   >=90  提示： xxx 分数为 90 成绩优秀
	   >=80  提示： xxx 分数为 80 成绩良好
	   >=60  提示： xxx 分数为 60 成绩及格
	   else  提示： xxx 分数为 5 成绩不及格
	   如果id对应的学生不存在，提示没有这个学生
*/
DELIMITER $
CREATE PROCEDURE pro_test4(sid INT,OUT des VARCHAR(20))
BEGIN
	DECLARE score1 INT;
	DECLARE name1 VARCHAR(20);
	
	SELECT score,NAME INTO score1,name1 FROM student WHERE id=sid;
	
	IF score1 IS NULL THEN
		SET des='没有这个学生';
	ELSEIF score1>=90 THEN
		SET des=CONCAT(name1,'分数为',score1,'成绩优秀');
	ELSEIF score1>=80 THEN
		SET des=CONCAT(name1,'分数为',score1,'成绩良好');
	ELSEIF score1>=60 THEN
		SET des=CONCAT(name1,'分数为',score1,'成绩及格');
	ELSE
		SET des=CONCAT(name1,'分数为',score1,'成绩不及格');
	END IF;
END$
DELIMITER ;

CALL pro_test4(3,@des);
SELECT @des;
-- ====================================================================================

/*
需求三：演示while，计算1-n的和
	n的值通过参数传递，传入的值小于1，提示参数不合法，结果输出为：1~n的和为：xx
*/
DELIMITER $
CREATE PROCEDURE pro_test5(N INT)
BEGIN
	-- 计数变量
	DECLARE num INT DEFAULT 1;
	-- 累计和变量
	DECLARE res INT DEFAULT 0;
	
	IF N<1 THEN
		SELECT '参数不合法';
	ELSE
		WHILE num<=N DO
			SET res = res + num;
			
			-- 控制条件
			SET num = num+1;
		END WHILE;
		
		-- 输出结果
		SELECT CONCAT('1~',N,'的和是：',res);
	END IF;

END$
DELIMITER ;
CALL pro_test5(10);

```

# 10、存储函数

存储函数：相当于是java中有返回值的方法,封装sql业务
   1.  方便sql业务的复用
   2. sql业务是事先编译好存储在数据库服务器端的，后续我们在执行操作时，就不需要在编译了，直接执行，相对而言效率更高

```mysql
创建：
delimiter $
    create function 函数名(参数列表)
    returns 数据类型
    begin
        sql业务;
        
        return 结果;
    end$
    delimiter ;
 调用:
select 函数名(参数值);

 参数列表:
构成：参数名 数据类型

-- 需求四：演示存储函数，获取student表中第2高的分数
DELIMITER $
CREATE FUNCTION fun_test1()
RETURNS INT
BEGIN
	DECLARE 2Height INT;
	SELECT DISTINCT score INTO 2Height FROM student ORDER BY score DESC LIMIT 1,1;
	RETURN 2Height;
END$
DELIMITER ;

SELECT fun_test1();
SELECT * FROM student WHERE score=fun_test1();
```

**综合案例：**

```mysql
/*
综合案例：获取第N高分数的学生信息，如果第N高的学生不存在，提示：没有该学生，如果存在，将第N高的学生信息返回
	（提示：通过存储函数获取第N高的分数，通过存储过程返回第N高分数的学生信息）
*/
DELIMITER $
CREATE FUNCTION fun_getNHScore(N INT)
RETURNS INT
BEGIN
	DECLARE nHeightScore INT;
	SET N = N-1;
	SELECT DISTINCT score INTO nHeightScore FROM student ORDER BY score DESC LIMIT N,1;
	RETURN nHeightScore;
END$
DELIMITER ;

DELIMITER $
CREATE PROCEDURE pro_getNHInfo(IN N INT)
BEGIN
	DECLARE nHScore INT;
	-- 获取第N高的学生分数
	SET nHScore = fun_getNHScore(N);
	IF nHScore IS NULL THEN
		SELECT '没有该学生';
	ELSE
		SELECT * FROM student WHERE score=nHScore;
	END IF;
	
END$
DELIMITER ;

CALL pro_getNHInfo(6);
```

# 11、触发器

触发器(了解)：监听表中记录的增删改操作，在这个操作发生之前或者发生之后，自动执行触发器中预先定义好的sql业务。

```mysql
   创建：
        delimiter $
        create trigger 触发器名
        before|after insert|update|delete
        on 表名
        for each row
        begin
		sql业务;(old获取操作之前的那一行数据;new获取操作之后的那一行数据)
        end$
        delimiter ;
```

# ==12、事务==
## 12.1  什么是事务？
#think **什么是事务？**

**事务: 保证一组相关联的增删改操作要么同时成功，要么同时失败的安全机制。**（同生共死；一荣俱荣，一损俱损）

经典案例：转账。
假如小明要给小红转账1000元，这个转账会涉及到两个关键操作就是：将小明的余额减少1000元，将小红的余额增加1000元。万一在这两个操作之间突然出现错误比如银行系统崩溃，导致小明余额减少而小红的余额没有增加，这样就不对了。事务就是保证这两个关键操作要么都成功，要么都要失败 。

## 12.2 操作步骤:(记忆)

```mysql
-- 开启事务
         start transaction
-- 执行关联的sql操作
         sql1
         sql2
         ...
-- 结束事务
         rollback  -- 同时失败
         commit  -- 同时成功
```

## 12.3 提交方式：(了解)

1. 自动提交：每一条sql语句执行完后，会将结果持久化保存到数据库的文件中，事务结束（mysql默认是自动提交）
2. 手动提交：每一条sql语句执行完后，修改只是数据库服务的内存中临时状态，一旦连接断开，数据会恢复，需要手动执行commit操作才会持久化保存（oracle默认是手动提交）

## 12.4 ==事务的特性:(记忆)==
#think  **事物的四大特性(ACID)?**
![300](img/Pasted%20image%2020211220151805.png)
1. **原子性：** 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用。
2.  **一致性：** 执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的；
3.  **持久性：** 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。
4.  **隔离性：** 并发访问数据库时，一个用户的事务不被其他事务所干扰，数据操作是隔离的；必须建立在一定的隔离级别之下

## 12.5 并发事务带来的读问题

#think  **什么是脏读？幻读？不可重复读？**

在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务（多个用户对统一数据进行操作）。并发虽然是必须的，但可能会导致以下的问题。
1.   **脏读（Dirty read）:** 当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。即一个事务读取到了另一个事务未提交的数据。

2.   **丢失修改（Lost to modify）:** 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。
	 - 例子：事务1读取某表中的数据A=20，事务2也读取A=20，事务1修改A=19，事务2也修改A=18，最终结果A=18，事务1的修改被丢失。
	
3.   **不可重复读（Unrepeatableread）:** 指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。
	   - 例子:小明多次读取A数据,小红在小明读取数据时,每次读取都修改了数据并提交,小明多次读到的数据不一致

4.  **幻读（Phantom read）:** 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。
	-   例子:小明修改了A,B数据,小红同时又插入了一条C数据,小明会感觉自己明明数据都改了,突然多出来一条,以为出现了幻觉自己漏了一条

**不可重复度和幻读区别：**

==不可重复读的重点是修改，幻读的重点在于新增或者删除。==

例1（同样的条件, 你读取过的数据, 再次读取出来发现值不一样了 ）：事务1中的A先生读取自己的工资为 1000的操作还没完成，事务2中的B先生就修改了A的工资为2000，导 致A再读自己的工资时工资变为 2000；这就是不可重复读。

例2（同样的条件, 第1次和第2次读出来的记录数不一样 ）：假某工资单表中工资大于3000的有4人，事务1读取了所有工资大于3000的人，共查到4条记录，这时事务2 又插入了一条工资大于3000的记录，事务1再次读取时查到的记录就变为了5条，这样就导致了幻读。

## 12.6  事务隔离级别

#think  **什么是事务的隔离级别？MySQL的默认隔离级别是什么？**

为了达到事务的四大特性，数据库定义了4种不同的事务隔离级别，由低到高依次为Read uncommitted、Read committed、Repeatable read、Serializable，这四个级别可以逐个解决脏读、不可重复读、幻读这几类问题。
![事务的隔离级别](img/Pasted%20image%2020211220150340.png)

**SQL 标准定义了四个隔离级别：**

1. **READ-UNCOMMITTED(读取未提交)：** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**。

2. **READ-COMMITTED(读取已提交)：** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**。

3. **REPEATABLE-READ(可重复读)：** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生**。

4. **SERIALIZABLE(可串行化)：** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。

**这里需要注意的是：Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别**
我们可以通过`SELECT @@tx_isolation;`命令来查看，MySQL 8.0 该命令改为`SELECT @@transaction_isolation;`

```shell
mysql> SELECT @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
```

>**MySQL InnoDB 的 REPEATABLE-READ（可重读）并不保证避免幻读，需要应用使用加锁读来保证。而这个加锁度使用到的机制就是 Next-Key Locks。**

事务隔离机制的实现基于锁机制和并发调度。其中并发调度使用的是MVVC（多版本并发控制），通过保存修改的旧版本信息来支持并发一致性读和回滚等特性。

因为隔离级别越低，事务请求的锁越少，所以大部分数据库系统的隔离级别都是**READ-COMMITTED(读取提交内容):**，但是你要知道的是InnoDB 存储引擎默认使用 **REPEATABLE-READ（可重读）**并不会有任何性能损失。

InnoDB 存储引擎在 **分布式事务** 的情况下一般会用到**SERIALIZABLE(可串行化)** 隔离级别。

>InnoDB 存储引擎提供了对 XA 事务的支持，并通过 XA 事务来支持分布式事务的实现。分布式事务指的是允许多个独立的事务资源（transactional resources）参与到一个全局的事务中。事务资源通常是关系型数据库系统，但也可以是其他类型的资源。全局事务要求在其中的所有参与的事务要么都提交，要么都回滚，这对于事务原有的 ACID 要求又有了提高。另外，在使用分布式事务时，InnoDB 存储引擎的事务隔离级别必须设置为 SERIALIZABLE。————《MySQL 技术内幕：InnoDB 存储引擎(第 2 版)》7.7 章

# 13、引擎
## 13.1 什么是引擎？
- jvm虚拟机：搭建java程序的运行环境，连接上层应用和底层系统硬件之间的桥梁
- 数据库引擎：搭建数据库的运行环境，连接数据库服务应用和底层文件系统之间的桥梁

存储引擎Storage engine：MySQL中的数据、索引以及其他对象是如何存储的，是一套文件系统的实现。

常用的存储引擎有以下：

· **Innodb引擎**：Innodb引擎提供了对数据库ACID事务的支持。并且还提供了行级锁和外键的约束。它的设计的目标就是处理大数据容量的数据库系统。

· **MyIASM引擎**(原本Mysql的默认引擎)：不提供事务的支持，也不支持行级锁和外键。

· **MEMORY引擎**：所有的数据都在内存中，数据的处理速度快，但是安全性不高。

#think **InnoDB引擎的4大特性**

 1. 插入缓冲（insert buffer)
 2. 二次写(double write)
 3. 自适应哈希索引(ahi)
 4. 预读(read ahead)

*分类：*
1. MyISAM:
	 - 表锁 -- 并发弱
	 - 不支持事务 -- 无法保证关联的增删改操作安全
	 - 不支持外键约束
	
2. InnoDB：
	- 表锁/行锁 -- 支持行锁，并发强
	- 支持事务 -- 可以保证关联的增删改操作安全
	- 支持外键约束

>在查询时，事务检查和外键约束检查会带来额外开销，所以查询操作InnoDB的效率会受到影响。

## 13.2 引擎操作
```mysql
引擎操作：（了解）
查看数据库支持的引擎
    show engines;
查看数据库中表的引擎
    show table status from 数据库名;
查看某一张表的引擎
    show table status from 数据库名 like '表名';
    show table status from 数据库名 where name = '表名';
创建表指定引擎
    create table 表名(...)engine=引擎名
修改表指定引擎
    alter table 表名 engine=引擎名;
```

 ## 13.3 存储引擎的区别及选择

#think **MySQL存储引擎MyISAM与InnoDB区别**

**MyISAM与InnoDB区别**
![](img/Pasted%20image%2020211220194012.png)
MySQL 5.5 之前，MyISAM 引擎是 MySQL 的默认存储引擎，可谓是风光一时。

虽然，MyISAM 的性能还行，各种特性也还不错（比如全文索引、压缩、空间函数等）。但是，MyISAM 不支持事务和行级锁，而且最大的缺陷就是崩溃后无法安全恢复。

5.5 版本之后，MySQL 引入了 InnoDB（事务性数据库引擎），MySQL 5.5 版本后默认的存储引擎为 InnoDB。

两者的主要区别在于以下几点：

**1.是否支持行级锁**

MyISAM 只有表级锁(table-level locking)，而 InnoDB 支持行级锁(row-level locking)和表级锁,默认为行级锁。

也就说，MyISAM 一锁就是锁住了整张表，这在并发写的情况下是多么糟糕！这也是为什么 InnoDB 在并发写的时候，性能更加厉害！

**2.是否支持事务**

MyISAM 不提供事务支持。

InnoDB 提供事务支持，具有提交(commit)和回滚(rollback)事务的能力。

**3.是否支持外键**

MyISAM 不支持，而 InnoDB 支持。

🌈 拓展一下：

一般我们也是不建议在数据库层面使用外键的，应用层面可以解决。不过，这样会对数据的一致性造成威胁。具体要不要使用外键还是要根据项目来决定。

**4.是否支持数据库异常崩溃后的安全恢复**

MyISAM 不支持，而 InnoDB 支持。

使用 InnoDB 的数据库在异常崩溃后，数据库重新启动的时候会保证数据库恢复到崩溃前的状态。这个恢复的过程依赖于 `redo log` 。

🌈 拓展一下：

-   MySQL InnoDB 引擎使用 **redo log(重做日志)** 保证事务的**持久性**，使用 **undo log(回滚日志)** 来保证事务的**原子性**。
-   MySQL InnoDB 引擎通过 **锁机制**、**MVCC** 等手段来保证事务的隔离性（ 默认支持的隔离级别是 **`REPEATABLE-READ`** ）。
-   保证了事务的持久性、原子性、隔离性之后，一致性才能得到保障。

**5.是否支持 MVCC**

MyISAM 不支持，而 InnoDB 支持。毕竟 MyISAM 连行级锁都不支持。

MVCC 可以看作是行级锁的一个升级，可以有效减少加锁操作，提供性能。

另外：
1. InnoDB索引是聚簇索引，MyISAM索引是非聚簇索引。
2.  InnoDB的主键索引的叶子节点存储着行数据，因此主键索引非常高效。
3.   MyISAM索引的叶子节点存储的是行数据地址，需要再寻址一次才能得到数据。
4.   InnoDB非主键索引的叶子节点存储的是主键和其他带索引的列数据，因此查询时做到覆盖索引会非常高效。

#think **存储引擎选择**

MyISAM：以读写插入为主的应用程序，比如博客系统、新闻门户网站。
Innodb：更新（删除）操作频率高，或者要保证数据的完整性；并发量高，支持事务和外键。比如OA自动化办公系统。
如果一个表只做查询操作，选择MyISAM；如果这个表涉及到增删改操作，选择InnoDB。
在我们的实际业务中基本没有表只做查询，所以对于引擎，基本就是选择默认的InnoDB；对于mysql服务系统的一些表，不提供增删改操作的话，会选择使用MyISAM

# ==14、索引==

 概念：索引是一种数据结构（通过某种方式，将数据组织在一起），在mysql数据库中默认选择的是B+Tree数据结构，通过索引可以提高查询数据的效率。

#think  **为什么选择使用B+Tree,而不是BTree结构?**

​      **答：B+Tree的每一个节点(页)包含更多的行记录信息,整个树结构相对于BTree而言,树的层级会更加浅。我们在查找数据时,需要读取的节点从次数(IO次数)就会更少,效率越高；另外当进行范围查询时，可以直接利用叶子节点之间的双向链表实现快速查询。**

```mysql
-- 小思考：
-- int类型的主键，大小是4byte，每一个节点(页)大小是16kb，一个节点大概可以包含1000个int类型主键值。对于这种索引列的值三层结构
-- 能表示多少行数据？大概是10亿级的数据体量，只要进行3次io操作就可以找到我们要的数据。  
```

相关概念:

- 磁盘块:底层系统存储数据的最小单元;但是mysql数据库在加载数据时,是以页为单位,每页默认是16kb,包含多个磁盘块，这一页也相当于是树上的一个节点。
- BTree:树中所有节点上都包含索引列的值和整行记录的数据,叶子节点之间是没有任何联系
- B+Tree:树中叶子节点包含整行记录的数据,但是非叶子节点只包含索引列的值,而且叶子节点之间通过双向关联的方式实现了双向链表结构

分类:

1. **聚簇索引:mysql所有数据的存储是直接存储在聚簇索引树上的,所以mysql说索引即数据,数据即索引。**

mysql数据库通过以下原则,保证一定会构建聚簇索引:
        ① 通过表的主键,构建聚簇索引
        ② 如果表没有主键,会选择一列唯一约束的的列,构建聚簇索引
        ③ 如果既没有主键,又没有唯一约束的列,会通过表中默认生成db_row_id列构建聚簇索引

**2.二级索引(辅助索引):这个索引树上只保存索引列的值和聚簇索引列的值,当我们查询数据时,如果通过这个树找不到我们需要的全部列,会通过聚簇索引列的值到聚簇索引上找,这个动作我们称之为"回表"。**
![回表](img/Pasted%20image%2020211220150828.png)

 ==**普通索引**==
 ==**联合索引**==
 唯一索引
 外键索引
 全文索引

**==索引操作==**：

```mysql

    创建索引:
        create index 索引名 on 表名(列名1,列名2,...);
        alter table 表名 add index 索引名(列名1,列名2,...);
        
    查看索引:
    show index from 表名;
    
	删除索引:
    drop index 索引名 on 表名;
    
-- 索引的创建原则：重要*****
    *** 索引的创建不是一劳永逸，需要根据后期的查询需求，做对应的取舍
    ① 一张表的索引个数不允许过多，不允许超过5个
      第一，每一个索引都是一个B+Tree的结构，需要占用数据库的存储空间，索引过多，会给存储带来压力
      第二，数据在索引树上是有序的，我们对表中记录进行增删改后，需要维护每个树的数据顺序，如果树过多，会造成增删改效率降低
      
    ② 根据查询条件，基于最左匹配原则，创建索引;最左匹配原则是适用于所有索引的，不仅限于联合索引
      数据在索引上是有顺序的，假设基于name和age构建一个联合索引
        数据是：zs-18 , zz-25, za-60
        这些数据在索引树上的顺序是：za-60,zs-18,zz-25
        
        select * from student where name='xx';  -- 可以使用索引
        select * from student where age=18;     -- 不可以使用索引
        select * from student where age=18 and name='xx'; -- 可以使用索引
        select * from student where age=18 or name='xx'  -- 不可以使用索引
        select * from student where name like 'xx%'     -- 可以使用索引
        select * from student where name like '%xx%'    -- 不可以使用索引
        
    ③ 索引列的值占据的空间要尽可能小，mysql每次加载一页（16kb）能包含的行记录信息会更多，我们可以通过更少的
	io操作来查询到我们想要的数据。实际开发中，如果就是需要按照某一列来构建索引，但是这一列的值又很大
	此时我们可以利用这一列的前几个字符来构建索引 -- “前缀索引”
	这个前缀到底要取多少个字符，需要进行计算的。
		SELECT COUNT(DISTINCT SUBSTRING(NAME,1,8))/COUNT(1) FROM product; -- 这个计算出来的值最起码大于90%
		CREATE INDEX idx_name ON product(NAME(8)); -- 基于name列的前8个字符，创建前缀索引

	④ 尽可能减少回表操作，可以将需要查询的列结合查询条件，一并构建联合索引，这种操作称之为“覆盖索引”
	CREATE INDEX idx_name ON product(NAME(8));
	SELECT id,NAME,price FROM product WHERE NAME='BfUifcYGBE'; -- 要查询price，需要回表
	
	CREATE INDEX idx_name_price ON product(NAME(8),price);
	SELECT id,NAME,price FROM product WHERE NAME='BfUifcYGBE'; -- 查询的price就在索引树上，不需要回表
```

代码示例：

```mysql
SELECT * FROM product WHERE id=500000;

SHOW TABLE STATUS FROM db4 WHERE NAME='product';
-- 查询索引
SHOW INDEX FROM product;
-- 创建普通索引
CREATE INDEX idx_price ON product(price);
-- 删除索引
DROP INDEX idx_price ON product;
-- 修改表创建索引
ALTER TABLE product ADD INDEX idx_name(NAME);

SELECT COUNT(*) FROM product;

SELECT * FROM product WHERE id=500000; -- BfUifcYGBE

SELECT * FROM product WHERE NAME='BfUifcYGBE';

-- 基于查询条件 name列创建一个索引
CREATE INDEX idx_name ON product(NAME);

SELECT * FROM product WHERE NAME='BfUifcYGBE';  -- 基于name索引查询，符合最左匹配原则

SELECT * FROM product WHERE NAME LIKE 'BfUif%'; -- 基于name索引查询，符合最左匹配原则

SELECT * FROM product WHERE NAME LIKE '%cYGBE'; -- 基于name索引查询，不符合最左匹配原则，整表扫描

DROP INDEX idx_name_price ON product;

-- 基于name和price构建联合索引
CREATE INDEX idx_name_price ON product(NAME,price);

SELECT * FROM product WHERE NAME='BfUifcYGBE' AND price=500000; -- 可以使用索引
SELECT * FROM product WHERE price=500000 AND NAME='BfUifcYGBE';  -- 可以使用索引
SELECT * FROM product WHERE price=500000 OR NAME='BfUifcYGBE';  -- 不可以使用索引
SELECT * FROM product WHERE price=500000 ;  -- 不可以使用索引


SELECT COUNT(DISTINCT SUBSTRING(NAME,1,8))/COUNT(1) FROM product;
-- 基于name列的前8个字符构建索引-- 前缀索引
CREATE INDEX idx_name ON product(NAME(8));

DROP INDEX idx_name ON product;
SHOW INDEX FROM product;

SELECT id,NAME,price FROM product WHERE NAME='BfUifcYGBE'; -- 需要回表

CREATE INDEX idx_name_price ON product(NAME(8),price);

SELECT id,NAME,price FROM product WHERE NAME='BfUifcYGBE'; -- 不需要回表


SELECT * FROM product WHERE id = 200 FOR UPDATE;
```

# 15、锁
## 15.1 锁的概念
 之前我们学习过多线程，多线程当中如果想保证数据的准确性是如何实现的呢？没错，通过同步实现。同步就相当于是加锁。加了锁以后有什么好处呢？当一个线程真正在操作数据的时候，其他线程只能等待。当一个线程执行完毕后，释放锁。其他线程才能进行操作！
    
那么我们的MySQL数据库中的锁的功能也是类似的。在我们学习事务的时候，讲解过事务的隔离性，可能会出现脏读、不可重复读、幻读的问题，当时我们的解决方式是通过修改事务的隔离级别来控制，但是数据库的隔离级别我们并不推荐修改。所以，锁的作用可以解决掉之前的问题！
    
**锁机制 :** 数据库为了保证数据的一致性，而使用各种共享的资源在被并发访问时变得有序所设计的一种规则。
    
-   举例，在电商网站购买商品时，商品表中只存有1个商品，而此时又有两个人同时购买，那么谁能买到就是一个关键的问题。
    
    这里会用到事务进行一系列的操作：
    
    1.  先从商品表中取出物品的数据
        
    2.  然后插入订单
        
    3.  付款后，再插入付款表信息
        
    4.  更新商品表中商品的数量
        
    
    以上过程中，使用锁可以对商品数量数据信息进行保护，实现隔离，即只允许第一位用户完成整套购买流程，而其他用户只能等待，这样就解决了并发中的矛盾问题。
    

在数据库中，数据是一种供许多用户共享访问的资源，如何保证数据并发访问的一致性、有效性，是所有数据库必须解决的一个问题，MySQL由于自身架构的特点，在不同的存储引擎中，都设计了面对特定场景的锁定机制，所以引擎的差别，导致锁机制也是有很大差别的。
锁

## 15.2锁的分类

1. *按操作分类：*
	1. **共享锁：也叫读锁。** 针对同一份数据，多个事务读取操作可以同时加锁而不互相影响 ，但是不能修改数据记录。
	2. **排他锁：也叫写锁。** 当前的操作没有完成前,会阻断其他操作的读取和写入
 2. *按粒度分类：*
	1. **表级锁：** 操作时，会锁定整个表。开销小，加锁快；不会出现死锁；锁定力度大，发生锁冲突概率高，并发度最低。偏向于MyISAM存储引擎！
	2. **行级锁：** 操作时，会锁定当前操作行。开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率低，并发度高。偏向于InnoDB存储引擎！
	3. **页级锁：** 锁的粒度、发生冲突的概率和加锁的开销介于表锁和行锁之间，会出现死锁，并发性能一般。
3. *按使用方式分类：*
	1. **悲观锁：** 每次查询数据时都认为别人会修改，很悲观，所以查询时加锁。
	2. **乐观锁：** 每次查询数据时都认为别人不会修改，很乐观，但是更新时会判断一下在此期间别人有没有去更新这个数据

- 不同存储引擎支持的锁

| 存储引擎 | 表级锁 | 行级锁 | 页级锁 |
| -------- | ------ | ------ | ------ |
| MyISAM   | 支持   | 不支持 | 不支持 |
| InnoDB   | 支持   | 支持   | 不支持 |
| MEMORY   | 支持   | 不支持 | 不支持 |
| BDB      | 支持   | 不支持 | 支持   |

## 15.3 演示InnoDB锁

- 数据准备

```mysql
-- 创建db13数据库
CREATE DATABASE db13;

-- 使用db13数据库
USE db13;

-- 创建student表
CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(10),
	age INT,
	score INT
);
-- 添加数据
INSERT INTO student VALUES (NULL,'张三',23,99),(NULL,'李四',24,95),
(NULL,'王五',25,98),(NULL,'赵六',26,97);
```

- 共享锁

```mysql
-- 标准语法
SELECT语句 LOCK IN SHARE MODE;
```

```mysql
-- 窗口1
/*
	共享锁：数据可以被多个事务查询，但是不能修改
*/
-- 开启事务
START TRANSACTION;

-- 查询id为1的数据记录。加入共享锁
SELECT * FROM student WHERE id=1 LOCK IN SHARE MODE;

-- 查询分数为99分的数据记录。加入共享锁
SELECT * FROM student WHERE score=99 LOCK IN SHARE MODE;

-- 提交事务
COMMIT;
```

```mysql
-- 窗口2
-- 开启事务
START TRANSACTION;

-- 查询id为1的数据记录(普通查询，可以查询)
SELECT * FROM student WHERE id=1;

-- 查询id为1的数据记录，并加入共享锁(可以查询。共享锁和共享锁兼容)
SELECT * FROM student WHERE id=1 LOCK IN SHARE MODE;

-- 修改id为1的姓名为张三三(不能修改，会出现锁的情况。只有窗口1提交事务后，才能修改成功)
UPDATE student SET NAME='张三三' WHERE id = 1;

-- 修改id为2的姓名为李四四(修改成功，InnoDB引擎默认是行锁)
UPDATE student SET NAME='李四四' WHERE id = 2;

-- 修改id为3的姓名为王五五(注意：InnoDB引擎如果不采用带索引的列。则会提升为表锁)
UPDATE student SET NAME='王五五' WHERE id = 3;

-- 提交事务
COMMIT;
```

- 排他锁

```mysql
-- 标准语法
SELECT语句 FOR UPDATE;
```

```mysql
-- 窗口1
/*
	排他锁：加锁的数据，不能被其他事务加锁查询或修改
*/
-- 开启事务
START TRANSACTION;

-- 查询id为1的数据记录，并加入排他锁
SELECT * FROM student WHERE id=1 FOR UPDATE;

-- 提交事务
COMMIT;
```

```mysql
-- 窗口2
-- 开启事务
START TRANSACTION;

-- 查询id为1的数据记录(普通查询没问题)
SELECT * FROM student WHERE id=1;

-- 查询id为1的数据记录，并加入共享锁(不能查询。因为排他锁不能和其他锁共存)
SELECT * FROM student WHERE id=1 LOCK IN SHARE MODE;

-- 查询id为1的数据记录，并加入排他锁(不能查询。因为排他锁不能和其他锁共存)
SELECT * FROM student WHERE id=1 FOR UPDATE;

-- 修改id为1的姓名为张三(不能修改，会出现锁的情况。只有窗口1提交事务后，才能修改成功)
UPDATE student SET NAME='张三' WHERE id=1;

-- 提交事务
COMMIT;
```

>注意：锁的兼容性
>    共享锁和共享锁     兼容
 > - 共享锁和排他锁     冲突
 > - 排他锁和排他锁     冲突
 > - 排他锁和共享锁     冲突
 > **读读共享/读写互斥/写写互斥**
  
## 15.4演示MyISAM锁

- 数据准备

```mysql
-- 创建product表
CREATE TABLE product(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20),
	price INT
)ENGINE = MYISAM;  -- 指定存储引擎为MyISAM

-- 添加数据
INSERT INTO product VALUES (NULL,'华为手机',4999),(NULL,'小米手机',2999),
(NULL,'苹果',8999),(NULL,'中兴',1999);
```

- 读锁

```mysql
-- 标准语法
-- 加锁
LOCK TABLE 表名 READ;

-- 解锁(将当前会话所有的表进行解锁)
UNLOCK TABLES;
```

```mysql
-- 窗口1
/*
	读锁：所有连接只能读取数据，不能修改
*/
-- 为product表加入读锁
LOCK TABLE product READ;

-- 查询product表(查询成功)
SELECT * FROM product;

-- 修改华为手机的价格为5999(修改失败)
UPDATE product SET price=5999 WHERE id=1;

-- 解锁
UNLOCK TABLES;
```

```mysql
-- 窗口2
-- 查询product表(查询成功)
SELECT * FROM product;

-- 修改华为手机的价格为5999(不能修改，窗口1解锁后才能修改成功)
UPDATE product SET price=5999 WHERE id=1;
```

- 写锁

```mysql
-- 标准语法
-- 加锁
LOCK TABLE 表名 WRITE;

-- 解锁(将当前会话所有的表进行解锁)
UNLOCK TABLES;
```

```mysql
-- 窗口1
/*
	写锁：其他连接不能查询和修改数据
*/
-- 为product表添加写锁
LOCK TABLE product WRITE;

-- 查询product表(查询成功)
SELECT * FROM product;

-- 修改小米手机的金额为3999(修改成功)
UPDATE product SET price=3999 WHERE id=2;

-- 解锁
UNLOCK TABLES;
```

```mysql
-- 窗口2
-- 查询product表(不能查询。只有窗口1解锁后才能查询成功)
SELECT * FROM product;

-- 修改小米手机的金额为2999(不能修改。只有窗口1解锁后才能修改成功)
UPDATE product SET price=2999 WHERE id=2;
```

## 15.5演示悲观锁和乐观锁

1. 悲观锁的概念

  - 就是很悲观，它对于数据被外界修改的操作持保守态度，认为数据随时会修改。
  - 整个数据处理中需要将数据加锁。悲观锁一般都是依靠关系型数据库提供的锁机制。
  - 我们之前所学的行锁，表锁不论是读写锁都是悲观锁。

2. 乐观锁的概念

  - 就是很乐观，每次自己操作数据的时候认为没有人会来修改它，所以不去加锁。
  - 但是在更新的时候会去判断在此期间数据有没有被修改。
  - 需要用户自己去实现，不会发生并发抢占资源，只有在提交操作的时候检查是否违反数据完整性。

- 悲观锁和乐观锁使用前提

  - 对于读的操作远多于写的操作的时候，这时候一个更新操作加锁会阻塞所有的读取操作，降低了吞吐量。最后还要释放锁，锁是需要一些开销的，这时候可以选择乐观锁。
  - 如果是读写比例差距不是非常大或者系统没有响应不及时，吞吐量瓶颈的问题，那就不要去使用乐观锁，它增加了复杂度，也带来了业务额外的风险。这时候可以选择悲观锁。

- 乐观锁的实现方式

  - 版本号

    - 给数据表中添加一个version列，每次更新后都将这个列的值加1。
    - 读取数据时，将版本号读取出来，在执行更新的时候，比较版本号。
    - 如果相同则执行更新，如果不相同，说明此条数据已经发生了变化。
    - 用户自行根据这个通知来决定怎么处理，比如重新开始一遍，或者放弃本次更新。

    ```mysql
    -- 创建city表
    CREATE TABLE city(
    	id INT PRIMARY KEY AUTO_INCREMENT,  -- 城市id
    	NAME VARCHAR(20),                   -- 城市名称
    	VERSION INT                         -- 版本号
    );
    
    -- 添加数据
    INSERT INTO city VALUES (NULL,'北京',1),(NULL,'上海',1),(NULL,'广州',1),(NULL,'深圳',1);
    
    -- 修改北京为北京市
    -- 1.查询北京的version
    SELECT VERSION FROM city WHERE NAME='北京';
    -- 2.修改北京为北京市，版本号+1。并对比版本号
    UPDATE city SET NAME='北京市',VERSION=VERSION+1 WHERE NAME='北京' AND VERSION=1;
    ```

  - 时间戳

    - 和版本号方式基本一样，给数据表中添加一个列，名称无所谓，数据类型需要是timestamp
    - 每次更新后都将最新时间插入到此列。
    - 读取数据时，将时间读取出来，在执行更新的时候，比较时间。
    - 如果相同则执行更新，如果不相同，说明此条数据已经发生了变化。

## 15.6锁的总结

1. 表锁和行锁
  - 行锁：锁的粒度更细，加行锁的性能损耗较大。并发处理能力较高。InnoDB引擎默认支持！
  - 表锁：锁的粒度较粗，加表锁的性能损耗较小。并发处理能力较低。InnoDB、MyISAM引擎支持！
2. InnoDB锁优化建议
  - 尽量通过带索引的列来完成数据查询，从而避免InnoDB无法加行锁而升级为表锁。
  - 合理设计索引，索引要尽可能准确，尽可能的缩小锁定范围，避免造成不必要的锁定。
  - 尽可能减少基于范围的数据检索过滤条件。
  - 尽量控制事务的大小，减少锁定的资源量和锁定时间长度。
  - 在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁产生概率。
  - 对于非常容易产生死锁的业务部分，可以尝试使用升级锁定颗粒度，通过表级锁定来减少死锁的产生。

**总结（InnoDB引擎）：**
一个事务基于索引列对某一个行加读锁，其他事务可以对这行做不加锁操作/加读锁操作；但是不可以对这一行做加写锁操作对其他行没有任何影响。

一个事务基于非索引列对某一个行加读锁，会给这个表上表锁，其他事务可以对这个表中的记录做不加锁操作/加读锁操作；但是不可以对这个表中的记录做加写锁操作。

一个事务基于索引列对某一个行加写锁，其他可以对这一行做不加锁操作，但是不可以做任何加锁操作对其他行没有任何影响。

一个事务基于非索引列对某一个行加写锁，会给这个表上表锁，其他事务可以对这个表中的记录做不加锁操作，不可以做任何加锁操作

## 15.7思考

#think **对MySQL的锁了解吗**

当数据库有**并发事务**的时候，可能会产生数据的不一致，这时候需要一些机制来保证访问的次序，锁机制就是这样的一个机制。

就像酒店的房间，如果大家随意进出，就会出现多人抢夺同一个房间的情况，而在房间上装上锁，申请到钥匙的人才可以入住并且将房间锁起来，其他人只有等他使用完毕才可以再次使用。

#think **隔离级别与锁的关系**

在Read Uncommitted级别下，读取数据不需要加共享锁，这样就不会跟被修改的数据上的排他锁冲突

在Read Committed级别下，读操作需要加共享锁，但是在语句执行完以后释放共享锁；

在Repeatable Read级别下，读操作需要加共享锁，但是在事务提交之前并不释放共享锁，也就是必须等待事务执行完毕以后才释放共享锁。

SERIALIZABLE 是限制性最强的隔离级别，因为该级别**锁定整个范围的键**，并一直持有锁，直到事务完成。

#think **按照锁的粒度分数据库锁有哪些？锁机制与InnoDB锁算法**

在关系型数据库中，可以**按照锁的粒度把数据库锁分**为行级锁(INNODB引擎)、表级锁(MYISAM引擎)和页级锁(BDB引擎 )。

**MyISAM和InnoDB存储引擎使用的锁：**

· MyISAM采用表级锁(table-level locking)。

· InnoDB支持行级锁(row-level locking)和表级锁，默认为行级锁

行级锁，表级锁和页级锁对比

**行级锁** 行级锁是Mysql中锁定粒度最细的一种锁，表示只针对当前操作的行进行加锁。行级锁能大大减少数据库操作的冲突。其加锁粒度最小，但加锁的开销也最大。行级锁分为共享锁 和 排他锁。

特点：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

**表级锁** 表级锁是MySQL中锁定粒度最大的一种锁，表示对当前操作的整张表加锁，它实现简单，资源消耗较少，被大部分MySQL引擎支持。最常使用的MYISAM与INNODB都支持表级锁定。表级锁定分为表共享读锁（共享锁）与表独占写锁（排他锁）。

特点：开销小，加锁快；不会出现死锁；锁定粒度大，发出锁冲突的概率最高，并发度最低。

**页级锁** 页级锁是MySQL中锁定粒度介于行级锁和表级锁中间的一种锁。表级锁速度快，但冲突多，行级冲突少，但速度慢。所以取了折衷的页级，一次锁定相邻的一组记录。

特点：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

#think **从锁的类别上分MySQL都有哪些锁呢？像上面那样子进行锁定岂不是有点阻碍并发效率了**

**从锁的类别上来讲**，有共享锁和排他锁。

共享锁: 又叫做读锁。 当用户要进行数据的读取时，对数据加上共享锁。共享锁可以同时加上多个。

排他锁: 又叫做写锁。 当用户要进行数据的写入时，对数据加上排他锁。排他锁只可以加一个，他和其他的排他锁，共享锁都相斥。

用上面的例子来说就是用户的行为有两种，一种是来看房，多个用户一起看房是可以接受的。 一种是真正的入住一晚，在这期间，无论是想入住的还是想看房的都不可以。

锁的粒度取决于具体的存储引擎，InnoDB实现了行级锁，页级锁，表级锁。

他们的加锁开销从大到小，并发能力也是从大到小。

#think **MySQL中InnoDB引擎的行锁是怎么实现的？**

InnoDB是基于索引来完成行锁

例: select * from tab_with_index where id = 1 for update;

for update 可以根据条件来完成行锁锁定，并且 id 是有索引键的列，如果 id 不是索引键那么InnoDB将完成表锁，并发将无从谈起

#think **InnoDB存储引擎的锁的算法有三种**

· Record lock：单个行记录上的锁

· Gap lock：间隙锁，锁定一个范围，不包括记录本身

· Next-key lock：record+gap 锁定一个范围，包含记录本身

**相关知识点：**

1. innodb对于行的查询使用next-key lock

2. Next-locking keying为了解决Phantom Problem幻读问题

3. 当查询的索引含有唯一属性时，将next-key lock降级为record key

4. Gap锁设计的目的是为了阻止多个事务将记录插入到同一范围内，而这会导致幻读问题的产生

5. 有两种方式显式关闭gap锁：（除了外键约束和唯一性检查外，其余情况仅使用record lock） A. 将事务隔离级别设置为RC B. 将参数innodb_locks_unsafe_for_binlog设置为1

#think **什么是死锁？产生死锁的四个条件？怎么解决？**

死锁是指两个或多个事务在同一资源上相互占用，并请求锁定对方的资源，从而导致恶性循环的现象。

*如果系统中以下四个条件同时成立，那么就能引起死锁：

1.   **互斥**：资源必须处于非共享模式，即一次只有一个进程可以使用。如果另一进程申请该资源，那么必须等待直到该资源被释放为止。
2.   **占有并等待**：一个进程至少应该占有一个资源，并等待另一资源，而该资源被其他进程所占有。
3.   **非抢占**：资源不能被抢占。只能在持有资源的进程完成任务后，该资源才会被释放。
4.   **循环等待**：有一组等待进程 `{P0, P1,..., Pn}`， `P0` 等待的资源被 `P1` 占有，`P1` 等待的资源被 `P2` 占有，......，`Pn-1` 等待的资源被 `Pn` 占有，`Pn` 等待的资源被 `P0` 占有。

>注意，只有四个条件==同时成立时，死锁才会出现。==

*常见的解决死锁的方法*

1、如果不同程序会并发存取多个表，尽量约定以相同的顺序访问表，可以大大降低死锁机会。

2、在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁产生概率；

3、对于非常容易产生死锁的业务部分，可以尝试使用升级锁定颗粒度，通过表级锁定来减少死锁产生的概率；

如果业务处理不好可以用分布式事务锁或者使用乐观锁

#think **数据库的乐观锁和悲观锁是什么？怎么实现的？**

数据库管理系统（DBMS）中的并发控制的任务是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性以及数据库的统一性。乐观并发控制（乐观锁）和悲观并发控制（悲观锁）是并发控制主要采用的技术手段。

**悲观锁**：假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作。在查询完数据的时候就把事务锁起来，直到提交事务。实现方式：使用数据库中的锁机制

**乐观锁**：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。在修改数据的时候把事务锁起来，通过version的方式来进行锁定。实现方式：乐一般会使用版本号机制或CAS算法实现。

**两种锁的使用场景**

从上面对两种锁的介绍，我们知道两种锁各有优缺点，不可认为一种好于另一种，像**乐观锁适用于写比较少的情况下（多读场景）**，即冲突真的很少发生的时候，这样可以省去了锁的开销，加大了系统的整个吞吐量。

但如果是多写的情况，一般会经常产生冲突，这就会导致上层应用会不断的进行retry，这样反倒是降低了性能，所以**一般多写的场景下用悲观锁就比较合适。**

# 16、优化
# 17、数据库集群