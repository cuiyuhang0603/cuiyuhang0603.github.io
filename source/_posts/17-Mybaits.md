---
title: 17-Mybatis笔记
date: 2021-11-04 20:00:00
tags: mybatis
---



# 1 概念

Mybatis：又名ibatis
	支持定制化SQL、存储过程以及高级映射的
	由Apache组织维护的开源的持久层框架
简单认知：替代jdbc

<!-- more -->

## jdbc的缺点

1. 频繁地创建和销毁连接：数据库服务器压力大
2. java代码和sql代码混合在一起：学习成本高 可读性差 维护麻烦
3. 给站位符赋值时：麻烦易错
4. 处理结果集时：麻烦易错

## mybaits解决jdbc的缺点

1. mybatis自带连接池，对连接的创建、初始化、分配、维护、销毁进行管理
2. mybatis是面向配置的编程：把sql语句从java代码中分离，sql映射文件
3. mybatis可以实现java对象与表记录的自动映射，
   	java对象的属性可以自动sql语句中的对应占位符赋值
4. mybatis可以实现表记录与java对象的自动映射，
   	可以实现数据行的数据自动解析为java对象

## mybatis的特点

1. 轻量级

2. 支持动态sql：通过标签实现sql的拼接

3. 面向配置的编程：核心配置文件(mybatis的基本设置)

   ​	sql映射文件(同一个表的所有sql标签)

 4. sql语句与Java代码的完全分离：单独的文件：sql映射文件存储所有的sql语句

 5. 良好支持复杂的数据映射：支持原生态的sql 比较适用于功能多变 模块不确定的项目

 6. 简单易学 容易上手

## mybatis和hibernate的区别

~~~
都是持久层框架：代替jdbc 核心都是ORM
1 mybatis简单易学 学习成本低
	hibernate比较复杂 学习成本高
2 hibernate是完全的orm框架：不存在sql 对jdbc进行了完全的封装
	mybatis不是完全的额orm框架，需要手写sql
3 hibernate比较适用于传统的 表结构固定的项目
	mybatis比较适用于功能模块多变 表结构复杂的项目
~~~



# 2 Mybatis基础

## 2.1 导入jar包

~~~
commons-loggin-1.1.1.jar
log4j-1.2.17.jar
mybatis-3.2.8.jar
mysql-connector-java-5.1.15-bin.jar
~~~

## 2.2 根据表创建实体类

> 注意：表的列名应当和类中的属性名一一对应



## 2.3 创建mybatis的核心配置文件

* mybatis_conf.xml(基础版本)

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
    
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
~~~

* dtd/xsd*

  > document type defined : 文档类型定义
  > 	给xml文件制定规范：制定xml的根标签+子标签+标签的属性



### 配置文件功能扩展

#### 1. 引入配置文件获取参数

~~~xml
<!-- 引入外部的配置文件获取数据库连接所需要的参数 -->
<properties resource="mybatis/db.properties" />
~~~

#### 2. 指定log4j日志系统

~~~xml
<settings>
    <!-- 指定log4j作为日志系统：显示sql语句 -->
    <setting name="logImpl" value="STDOUT_LOGGING" />
</settings>
~~~

#### 3. 给实体类起别名

~~~xml
<!-- 给实体类起别名，在sql的xml映射文件中可以直接使用别名 -->
<typeAliases>
     <!-- 方式1：可以直接给单独的类起别名 -->
    <typeAlias type="mybatis.Student" alias="stu"/>
    
    <!-- 方式2：可以给实体类所在包引入，就可以直接使用包中的类名，不用加包名了 -->
    <package name="mybatis" />    
</typeAliases>
~~~

* 完整版

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!-- 注意书写配置标签的顺序， 外部的数据源文件比较靠前， 其次是起别名的配置标签 -->
	<!-- 引入外部的配置文件获取数据库连接所需要的参数 -->
	<properties resource="mybatis/db.properties" />
	
	<settings>
		<!-- 指定log4j作为日志系统：显示sql语句 -->
		<setting name="logImpl" value="STDOUT_LOGGING" />
	</settings>

	<!-- 给实体类起别名，在sql的xml映射文件中可以直接使用别名 -->
	<typeAliases>
		<!-- 方式2：可以给实体类所在包引入，就可以直接使用包中的类名，不用加包名了 -->
		<package name="mybatis" /> 
		
		<!-- 方式1：可以直接给单独的类起别名 -->
		<!-- <typeAlias type="mybatis.Student" alias="stu"/> --> 
	</typeAliases>

    <!-- 在导入了外部配置文件properties之后，可以通过变量的形式获取数据库链接参数 -->
	<environments default="datasource">
		<environment id="datasource">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
	</environments>

	<mappers>
		<!-- 配置sql语句的映射xml文件，路径相对src -->
		<mapper resource="mybatis/StudentMapper.xml" />
	</mappers>
</configuration>
~~~



## 2.4 创建sql的mapper文件

* StudentMapper.xml

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 命名空间用来映射接口，将所有的sql语句映射为接口的方法 -->
<mapper namespace="/">
    
  <select id="getOne" resultType="包名.类名" parameterType="int">
      select * from student where sid = #{id}
  </select>
  <delete id="" parameterType=""></delete>
  <update id="" parameterType=""></update>
  <insert id="" parameterType=""></insert>
    <!-- 注：mapper不能有空的sql标签语句，否则可能会报错 -->
    
</mapper>
<!-- 每一种sql语句对应一种标签	
	mybatis的sql配置文件中：#{xxx}是占位符
	通用属性：
		id:	用来调用该标签语句的别名 标签的id属性必须唯一
 		parameterType:	参数的类型(list和单个对象相同)
	注：当parameterType是单值类型时：#{}占位符中的名字 可以随意
		当parameterType是对象类型时：占位符中的名字必须和对象的属性名一致
	
	select标签的属性：
        resultType：结果集中"每行"对应的java对象的类型
 -->
~~~

### 2.4.1 多个参数传递问题

* 方案1：将数据封装为一个对象

~~~xml
<!-- 
   Object obj = new Objec(参数1,参数2,...);
      dao.getOne(obj);-->
<select id="getOne" restultType="" paramType="Object">
    select * from table where #{obj.参数1} and #{obj.参数2} ...
</select>
~~~

* 方案2：将数据封装为一个集合

  * List封装：#{list[下标] (.属性名) }

  * map封装：#{键}

    > 使用map封装数据，站位符需要和键名一致

  * 数组封装：#{array[下标] (.属性名)}

~~~xml
<select id="getOne" restultType="" paramType="map">
    select * from table where #{键值名} and #{键值名} ...
</select>
~~~

* 方案3：直接使用索引：从0开始

  * #{0}：对应参数列表的第一个参数

~~~xml
<select id="getOne" restultType="">
    select * from table where #{0} and #{1} ...
</select>
~~~

* 方案4：使用`@param("别名")`注解为参数起别名

  * #{别名}：对应添加注解的参数

~~~java
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(new FileInpuStream(new File("path")));
new Dao().getOne(@param("别名1")参数1,@param("别名2")参数2,..);
~~~

~~~xml
<select id="getOne" restultType="">
    select * from table where #{别名1} and #{别名2},...
</select>
~~~

### 2.4.2 自定义列名、属性映射

> 问题描述：数据表的列名和对象属性名不一致时，无法进行自动映射

* 解决方案1：在sql语句中为结果集起别名

~~~xml
<!-- 结果集列明与对象属性名不一致：解决方案1：给结果集起别名： -->
<!-- List<Student> getAll2(); -->
<select id="getAll2" resultType="Student">
    select id sid,name sname,score,sex,dy sdy,tid stid,age sage from tab_stu
</select>
~~~

* 解决方案2：定义resultMap指定列名和对象属性的对应关系

~~~xml
<!-- List<Student> getAll3(); -->
<resultMap type="Student" id="stuMap3">
    <id column="id"  property="sid" javaType="int"/>
    <result column="name" property="sname" javaType="String"/>
    <result column="age" property="sage" javaType="int"/>
    <result column="dy" property="sdy" javaType="boolean"/>
    <result column="tid" property="stid"/>
    <!-- 本来属性名和列名一致的可以不用设置，顺序也可打乱
        <result column="sex" property="sex"/>
          <result column="score" property="score"/>
       -->
</resultMap>
<select id="getAll3" resultMap="stuMap3">
    select * from tab_stu
</select>
~~~



## 2.5 关联sql的mapper文件

* mybatis_conf.xml

~~~xml
<!-- 引入所有的sql映射文件 -->
<mappers>
	<mapper resource="包名/包名/.../文件名" />
</mappers>
<!-- 路径相对于src，文件路径之间使用/分割符 -->
~~~



## 2.6 使用方式

### 2.6.1 原始dao方式

~~~java
// 1 获取sqlsessionfactorybuilder对象
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
// 2 slqsessionfactorybuilder读取核心配置文件 获取sqlsessionfactory对象
File file = new File("src/mybatis/mybatis_config.xml");

SqlSessionFactory factory = builder.build(new FileInputStream(file));// 抛出异常
// 3 获取sqlsession：类似于connection连接
SqlSession session = factory.openSession(true);// boolean表示是否支持事物自动提交

// 只有dml才有事物：多个dml组成一个整体 同生共死
// 4 通过sqlsession的方法调用sql标签 访问数据库
Student s1 = session.selectOne("getOne", 997);

// session.commit();//事务提交
// 5 关闭session
session.close();
~~~

### 2.6.2 mapper代理方式

~~~java
//其他步骤与上述无区别

// 4 通过sqlsession的方法创建映射接口的代理类
StudentMapper studentMapper = session.getMapper(StudentMapper.class);
// 4.1 调用接口的方法，会执行sql的xml映射文件中的sql语句
Student student = studentMapper.getOne(997);
~~~



# 3 Mybatis进阶

结合Mybatis基础的配置文件功能扩展进行理解

## 3.1 配置日志系统

以配置log4h日志文件为例：

1. 导入jar包：common-logging-1.1.1.jar + log4j-1.2.17.jar

2. 创建一个log4j的核心配置文件：名字必须是log4j.properties (<font color=red>位置必须是src</font>)

~~~
log4j.logger.com.ibatis=DEBUG
log4j.logger.com.ibatis.common.jdbc.SimpleDataSource=DEBUG
log4j.logger.com.ibatis.common.jdbc.ScriptRunner=DEBUG
log4j.logger.com.ibatis.sqlmap.engine.impl.SqlMapClientDelegate=DEBUG
log4j.logger.Java.sql.Connection=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
~~~

3. 在mybatis核心配置文件中添加setting标签

~~~xml
<settings>
    <!-- 制定log4j作为日志系统：显示sql语句 -->
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
~~~

* 使用效果

~~~
log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Logging initialized using 'class org.apache.ibatis.logging.stdout.StdOutImpl' adapter.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
Opening JDBC Connection
Created connection 214074868.
==>  Preparing: select * from student where sid = ? 
==> Parameters: 997(Integer)
<==    Columns: sid, sname, sex, score, stid, sage, sdy
<==        Row: 51, 新xxx, 仙, 111.0, 2, 12, 1
<==      Total: 1
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@cc285f4]
Returned connection 214074868 to pool.
~~~

## 3.2 日志log4j详解

* 日志系统

~~~
系统日志是记录系统中硬件、软件和系统问题的信息，同时还可以还可以见识系统中发生的事情，
对用户操作进行统计。。。
分类：服务器日志、用户操作日志、错误日志、数据库日志
~~~

* log4j

~~~
log for java 为java写的一个日志系统
~~~

使用步骤：

1. 导入jar包：log4j-1.2.17.jar

2. 在创建log4j的核心配置文件

   * 位置必须是src
   * 名字必须是log4j.properties

~~~properties
#日志级别
#A：off 最高等级，用于关闭所有日志记录。
#B：fatal 指出每个严重的错误事件将会导致应用程序的退出。
#C：error 指出虽然发生错误事件，但仍然不影响系统的继续运行。
#D：warm 表明会出现潜在的错误情形。
#E：info 一般和在粗粒度级别上，强调应用程序的运行全程。
#F：debug 一般用于细粒度级别上，对调试应用程序非常有帮助。
#G：all 最低等级，用于打开所有日志记录。
# 指定统一的日志级别为INFO
# 指定两个记录官：stdout , R
log4j.rootCategory=INFO, stdout , R

#记录官stdout：类型是ConsoleAppender--要把信息打印到控制台
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
#记录官stdout：日志信息格式是自定义的
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
#记录官stdout：日志信息格式
log4j.appender.stdout.layout.ConversionPattern=[QC] %p [%t] %C.%M(%L) | %m%n

#记录官stdout：类型是DailyRollingFileAppender--每天生成一个新的文件
log4j.appender.R=org.apache.log4j.DailyRollingFileAppender
log4j.appender.R.File=E:\\log4.log
log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=%d-[TS] %p %t %c - %m%n

log4j.logger.com.neusoft=DEBUG
log4j.logger.com.opensymphony.oscache=ERROR
log4j.logger.net.sf.navigator=ERROR
log4j.logger.org.apache.commons=ERROR
log4j.logger.org.apache.struts=WARN
log4j.logger.org.displaytag=ERROR
log4j.logger.org.springframework=DEBUG
log4j.logger.com.ibatis.db=WARN
log4j.logger.org.apache.velocity=FATAL

log4j.logger.com.canoo.webtest=WARN

log4j.logger.org.hibernate.ps.PreparedStatementCache=WARN
log4j.logger.org.hibernate=DEBUG
log4j.logger.org.logicalcobwebs=WARN
~~~

   

## 3.3 读取配置文件

* 在java中后缀为`.properties`的文件为配置文件
  * 格式：属性=属性名
  * 作用：作为配置文件，数据之间没有关系；只有单一的名字+值这样类型的数据

* 解析属性集文件

~~~java
//创建properties对象
Properties pros = new Properties();
//关联属性集文件
File file = new File("src/...");//文件路径相对项目
pros.load(new FileInputStream(file));
//获取属性值
pros.getProperty("jdbc.driver");
pros.getProperty("jdbc.username");
...

//遍历：和遍历hashmap相同
//获取素有的键值对应的set 通过键获取值
Set<Object> keys = pros.keySet();

//获取所有键值对对象对应的set 通过Entry的getKey和getValue获取键和值
Set<Entry<Object,Object>> entry = pros.entrySet();
~~~



## 3.4 复杂数据查询

在对应数据库中的多表联查，将不同的表中的数据拼凑成一个新表，
对应java代码中的对象嵌套，将拼凑的新表数据映射到嵌套的对象中，
这就是本部分主要讲解的部分

### 3.4.1 一对一的结果集映射

> 情景说明：涉及到多表查询，两张表的两行数据之间有外键的索引关系，
> 需要通过级联查询获取，两者的数据；在java对象中，表现为对象的属性
> 为引用类型，为其他类型的实体类对象

~~~java
/*
    学生成绩类，其中包含了学生的一科成绩信息，
    学号引用学生，课程号引用课程	
*/
public class StudentScore implements Serializable{
	private Integer scSid;
	private Integer scCid;
	private Float scScore;
    
	//外键引用学生的信息
	private Student student;
}
public class Student implements Serializable{
	private Integer sSid;
	private String sName;
	private Integer sAge;
	private Character sSex;
}
~~~

* 方案1：子查询

~~~xml
<!-- 普通单一查询 -->
<resultMap type="student" id="stu">
    <id column="s_sid" property="sSid"/>
    <result column="s_name" property="sName"/>
    <result column="s_age" property="sAge"/>
    <result column="s_sex" property="sSex"/>
</resultMap>
<select id="getOne" resultMap="stu" parameterType="int">
    select * from student where s_sid = #{id}
</select>
...
<!-- 1对1：使用子查询方式 -->
<resultMap type="studentScore" id="sc1">
    <id column="sc_sid" property="scSid"/>
    <result column="sc_cid" property="scCid"/>
    <result column="sc_score" property="scScore"/>
    <association property="student" column="sc_sid" 	
                 select="mybatis.dao.StudentMapper.getOne"/>
</resultMap>
<select id="getOne1" resultMap="sc1">
    select * from sc where sc_sid=#{0} and sc_cid=#{1}
</select>
~~~

* 方案2：结果集映射resultMap

~~~xml
<!-- 1对1：使用resultMap套接 -->
<resultMap type="studentScore" id="sc2">
    <id column="sc_sid" property="scSid"/>
    <result column="sc_cid" property="scCid"/>
    <result column="sc_score" property="scScore"/>
    <association property="student" 
                 resultMap="mybatis.dao.StudentMapper.stu"/>
</resultMap>
<select id="getOne2" resultMap="sc2">
    select * from sc,student as s where sc.sc_sid=s.s_sid and sc.sc_sid=#{0} and sc.sc_cid=#{1}
</select>
~~~



### 3.4.2 一对多的结果集映射

> 情景说明：和多对一的情景相似，但是引用类型为多个对象，是个集合

~~~java
public class Student implements Serializable{
	private Integer sSid;
	private String sName;
	private Integer sAge;
	private Character sSex;
	
	//学生类中包含该学生的所有成绩列表
	private List<StudentScore> sc;
}
public class StudentScore implements Serializable{
	private Integer scSid;
	private Integer scCid;
	private Float scScore;
}
~~~

* 方案1：子查询

~~~xml
<!-- 普通列表查询查询  -->
<resultMap type="studentScore" id="sc">
    <id column="sc_sid" property="scSid"/>
    <result column="sc_cid" property="scCid"/>
    <result column="sc_score" property="scScore"/>
</resultMap>
<select id="getList" resultMap="sc" parameterType="int">
    select * from sc where sc_sid=#{0}
</select>
...
<!-- 1对n查询：使用子查询的方式 -->
<resultMap type="student" id="stu1">
    <id column="s_sid" property="sSid"/>
    <result column="s_name" property="sName"/>
    <result column="s_age" property="sAge"/>
    <result column="s_sex" property="sSex"/>
    <collection property="sc" column="s_sid" 
                select="mybatis.dao.StudentScoreMapper.getList" />
</resultMap>
<select id="getOneAndList1" resultMap="stu1" parameterType="int">
    select * from student where s_sid = #{id}
</select>
~~~

* 方案2：结果集映射resultMap

~~~xml
<!-- 1对n查询：使用resultMap套接连接查询结果 -->
<resultMap type="student" id="stu2">
    <id column="s_sid" property="sSid"/>
    <result column="s_name" property="sName"/>
    <result column="s_age" property="sAge"/>
    <result column="s_sex" property="sSex"/>

    <collection property="sc" ofType="studentScore">
        <result column="sc_sid" property="scSid" />
        <result column="sc_cid" property="scCid" />
        <result column="sc_score" property="scScore" />
    </collection>
</resultMap>
<select id="getOneAndList2" resultMap="stu2" parameterType="int">
    select * from student s,sc where sc.sc_sid=s.s_sid and s.s_sid = #{id}
</select>
~~~

### 3.4.3 多对多的结果集映射

> 情景说明：和一对多的情景相似，查询结果是个对象集合，且集合中的对象包含了一个集合

~~~java
public class Student implements Serializable{
	private Integer sSid;
	private String sName;
	private Integer sAge;
	private Character sSex;
	
	//学生类中包含该学生的所有成绩列表
	private List<StudentScore> sc;
}
public class StudentScore implements Serializable{
	private Integer scSid;
	private Integer scCid;
	private Float scScore;
}
~~~

* 方案1：子查询

~~~xml
<!-- 普通列表查询查询  -->
<resultMap type="studentScore" id="sc">
    <id column="sc_sid" property="scSid"/>
    <result column="sc_cid" property="scCid"/>
    <result column="sc_score" property="scScore"/>
</resultMap>
<select id="getList" resultMap="sc" parameterType="int">
    select * from sc where sc_sid=#{0}
</select>
...
<!-- n对n查询：使用子查询的方式 -->
<resultMap type="student" id="stu11">
    <id column="s_sid" property="sSid"/>
    <result column="s_name" property="sName"/>
    <result column="s_age" property="sAge"/>
    <result column="s_sex" property="sSex"/>
    <collection property="sc" column="s_sid" 
                select="mybatis.dao.StudentScoreMapper.getList"/>
</resultMap>
<select id="getAllAndList1" resultMap="stu11" parameterType="int">
    select * from student s,sc where sc.sc_sid=s.s_sid
</select>
~~~

* 方案2：结果集映射resultMap

~~~xml
<!-- n对n查询：使用resultMap套接连接查询结果 -->
<resultMap type="student" id="stu12">
    <id column="s_sid" property="sSid"/>
    <result column="s_name" property="sName"/>
    <result column="s_age" property="sAge"/>
    <result column="s_sex" property="sSex"/>

    <!-- 注意使用id标签会对list集合进行去重 -->
    <collection property="sc" ofType="studentScore">
        <result column="sc_sid" property="scSid" />
        <result column="sc_cid" property="scCid" />
        <result column="sc_score" property="scScore" />
    </collection>
</resultMap>
<select id="getAllAndList2" resultMap="stu12" parameterType="int">
    select * from student s,sc where sc.sc_sid=s.s_sid
</select>
~~~



## 3.5 使用注解（[参考网址](https://zhuanlan.zhihu.com/p/163672002)）

> 注意：使用注解时，也是需要指定mapper.xml文件的！！！

```text
基本的CRUD头
 @Insert()
 @Delete()
 @Update() 
 @Select()

 @Results    ==> resultMap
 @Result     ==> result
 @One        ==  association
 @Many       ==> collection
```

* 简单语句

~~~java
@Insert("insert into user values(#{id},#{username})")
public void addUser(User user);

@Delete("delete from user where id=#{id}")
public void deleteUser(Integer id);

@Update("update user set username=#{username} where id=#{id}")
public void updateUser(User user);

@Select("select * from user")
public List<User> selectUser();
~~~

* 复杂语句

~~~java
//一对一查询
@Select("select * from orders")
@Results({
    @Result(property = "id",column = "id"),
    @Result(property = "orderTime",column = "orderTime"),
    @Result(property = "total",column = "total"),
    @Result(property = "user",column = "uid",javaType = User.class,
            //这里 的select="命名空间.方法名称"
            one=@One(select = "com.lagou.mapper.IUserMapper.findById"))
})
public List<Order> findAll();
~~~

~~~java
//一对多查询
//因为这里是一对多的关系,所以这里的javaType的类型为List.class
//然后这里不能使用@one 而是@many，表示这里有多个集合
@Select("select * from user")
@Results({
    @Result(property = "id",column = "id"),
    @Result(property = "username",column = "username"),
    @Result(property = "orderList",column = "id",javaType = List.class,
            //这里 的select="命名空间.方法名称"
            many = @Many(select = "com.lagou.mapper.IOrderMapper.findOrderByUid")),
})
public List<User> findAllUser();
~~~

~~~java
//多对多
@Select("select * from user")
@Results({
    @Result(property = "id",column = "id"),
    @Result(property = "username",column = "username"),
    @Result(property = "orderList",column = "id",javaType = List.class,
            many = @Many(select = "com.lagou.mapper.IOrderMapper.findOrderByUid")),
})
public List<User> findAllUser();

@Select("select * from user")
@Results({
    @Result(property = "id",column = "id"),
    @Result(property = "username",column = "username"),
    @Result(property = "roleList",column = "id",javaType = List.class,
            many = @Many(select = "com.lagou.mapper.IRoleMapper.findByUserId")),
})
public List<User> findAllUserAndRoles();


//和一对多不同的是，这里的sql语句是
@Select("select * from sys_role r,sys_user_role ur where r.id=ur.roleid and ur.userid=#{uid}")
public List<Role> findByUserId(Integer uid);
~~~



## 3.6 动态sql

* if标签

~~~xml
<if test="sex != null">
    and sex-#{sex}
</if>
<!-- 用法介绍：
	当test中的条件成立时，就将语句拼接在指定位置
注意1：该标签中可以引用参数，
	若参数是对象，直接使用对象属性名
	若是多值参数，可以使用param1的形式，按照参数顺序引用
注意2：外部使用单引号，则内部使用双引号；
	内部使用双引号，则外部使用单引号；
可以使用运算符：和jstl的if标签类似
-->
~~~

* where标签

~~~xml
<where>
    <if test="score != null and score gte 0 and score lte 100">
        score >=#{score}
    </if>
</where>
<!-- 用法介绍：
	动态拼接where关键字，当标签内有sql语句时，添加where
注意：可以去除第一个条件出现的前置and关键字，
		但是不能去除结尾
一般用于select标签内部
-->

~~~

* set标签

~~~xml
<set>
    <if test="sname != null">
        sname=#{sname},
    </if>
    <if test="sdy != null">
        sdy=#{sdy},
    </if>
</set>
<!-- 用法介绍：
	可以动态拼接多个赋值语句，并去除掉最后的逗号
注意：只适用于update语句
 -->
~~~

* choose标签

~~~xml
<choose>
    <when test="sex != null">
        sex=#{sex}
    </when>
    <when test="sdy != null">
        sdy=#{sdy}
    </when>
    <otherwise>
        stid=1 
    </otherwise>
</choose>
<!-- 用法介绍：
	类似java中的switch分支，多条分支只取最先成立的一条
注意：使用时注意条件的先后顺序
 -->
~~~

* trim标签

~~~xml
<trim prefix="where" 
      prefixOverrides="and | or"  
      suffix=" and 1=1 ">
    <if test="sex != null">
        and sex=#{sex}
    </if>
    <if test="score != null">
        or score=#{score}
    </if>
</trim>
<!-- 用法介绍：
	用于对整段sql语句的前后端进行处理，修改前后缀
	prefix:整体添加前缀
	suffix：整体添加后缀
	prefixoverrides：去除指定前缀
	suffixoverrides：去除指定后缀
 -->
~~~

* foreach标签

~~~xml
<foreach collection="array" 
         item="id" 
         separator="," 
         open="(" index="i" close=")" >
    #{id}
</foreach>
<!-- foreach标签 : 变量集合
    collection：指定集合的类型：array/list
    item: 定义变量记录集合中的元素
    index：	遍历时元素的下标
    
	separator：每个元素之间的分隔符
    open:	整体前面加的前缀
    close:	整体前面加的后缀
-->
~~~

* bind标签

~~~xml
<bind name="sn" value="'%' + _parameter + '%'"/>
<!-- 用法介绍：
	可以在sql标签内部使用，自定义变量
	一般用于对参数变量进行加工处理，比如左右添加%符号
 -->
~~~

* sql标签

~~~xml
<sql id="">sql语句</sql>
<!-- 用法介绍:
	一般用于复用sql可以直接用来拼接
 -->
~~~

### 3.6.2 标签中引用参数、参数的属性

1. 参数是对象：通过属性名来获取参数的值
2. 参数是多个单值数据：通过paramx来获取第x个参数的值
3. 参数是一个单值数据：通过_parameter获取此唯一的参数数据



# 4 逆向工程

> 概念：通过程序，由表(关系模型)直接生成实体类(与模型)及其相关代码由实体类自动生成表及其关系

## 4.1 使用流程

1. 创建项目

   > 专门有一个项目 用于测试和方向工程，一个项目一般只用一次即可，
   > 所以不必加入到项目中。

2. 导入jar包

   * jdbc
   * mybatis
   * log4j
   * mybatis-generator-core-1.3.2.jar

3. 创建逆向工程的配置文件：generatorConfig.xml

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >  
<generatorConfiguration>   
    <!-- 一个数据库一个context -->  
    <context id="infoGuardian">  
        <!-- 注释 -->  
        <commentGenerator >  
            <property name="suppressAllComments" value="true"/><!-- 生成代码的时候是否生成注释，true是取消注释，false会生成注释 -->  
            <property name="suppressDate" value="true" /> <!-- 是否生成注释代时间戳-->  
        </commentGenerator>  

        <!-- jdbc连接 -->  
        <jdbcConnection 
                        driverClass="com.mysql.jdbc.Driver"  
                        connectionURL="jdbc:mysql://localhost:3306/db_11" 
                        userId="root"  
                        password="root" />  

        <!-- 类型转换 -->  
        <javaTypeResolver>  
            <!-- 默认为false,可以把数据库中的decimal以及numeric类型解析为Integer，为true时会解析为java.math.BigDecimal） -->  
            <property name="forceBigDecimals" value="false"/>  
        </javaTypeResolver>  

        <!-- 生成实体类地址 -->    
        <javaModelGenerator targetPackage="com.zhiyou100.entity"  
                            targetProject=".\src" >  
            <!-- 是否在当前路径下新加一层schema,
如果为fase路径com.shop.pojo， 为true:com.shop.pojo.[schemaName]  这个情况主要是oracle中有，mysql中没有schema -->  
            <property name="enableSubPackages" value="false"/>  
            <!-- 是否针对string类型的字段在set的时候进行trim调用 -->  
            <property name="trimStrings" value="true"/>  
        </javaModelGenerator>  

        <!-- 生成mapxml文件 -->  
        <sqlMapGenerator targetPackage="com.zhiyou100.mapper"  
                         targetProject=".\src" >  
            <!-- 是否在当前路径下新加一层schema,
如果为fase路径com.shop.dao.mapper， 为true:com.shop.dao.mapper.[schemaName]  这个情况主要是oracle中有，mysql中没有schema -->  
            <property name="enableSubPackages" value="false" />  
        </sqlMapGenerator>  

        <!-- 生成mapxml对应client，也就是接口dao -->      
        <javaClientGenerator targetPackage="com.zhiyou100.dao"  
                             targetProject=".\src" type="XMLMAPPER" >  
            <!-- 是否在当前路径下新加一层schema,
如果为fase路径com.shop.dao.mapper， 为true:com.shop.dao.mapper.[schemaName]  这个情况主要是oracle中有，mysql中没有schema -->  
            <property name="enableSubPackages" value="false" />  
        </javaClientGenerator>  

        <!-- 配置表信息 -->      
        <table schema="" tableName="student"  
               domainObjectName="Student" enableCountByExample="false"  
               enableDeleteByExample="false" enableSelectByExample="false"  
               enableUpdateByExample="false">  
            <!-- schema即为数据库名 tableName为对应的数据库表, domainObjectName是要生成的实体类的名字， enable*ByExample指的是否生成 对应的example类，Mybatis Generator默认设置会生成一大堆罗哩罗嗦的Example类,主要是用各种不同的条件来操作数据库,大部分是用不到的,用到的时候手工修改mapper和接口文件就行了   -->  

            <!-- 忽略列，不生成bean 字段 -->  
            <ignoreColumn column="stid" />  
            <!-- 指定列的java数据类型
            <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" /> 
              -->  
        </table>  

    </context>  
</generatorConfiguration>  
~~~

4. 创建启动类

~~~java
public static void main(String[] args)throws Exception {
    List<String> warnings = new ArrayList<String>();
    boolean overwrite = true;
    File configFile = new File("src/generatorConfig.xml"); 
    ConfigurationParser cp = new ConfigurationParser(warnings);
    Configuration config = cp.parseConfiguration(configFile);
    DefaultShellCallback callback = new DefaultShellCallback(overwrite); 
    MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings); 
    myBatisGenerator.generate(null);
}
~~~



# 5 缓存

> 概念：当对数据库进行查询时，如果两次的sql语句一致，不再重复去服务数据库
> 直接使用上一次的查询结果
>
> 作用：提高程序效率+降低数据库服务器的压力
>
> [参考网址](https://www.cnblogs.com/happyflyingpig/p/7739749.html)

## 5.1 一级缓存

> 概念：Mybatis对缓存提供支持，一级缓存默认开启，无法关闭
>
> 现象：在参数和SQL完全一样的情况下，使用同一个sqlSession对象调用一个
> Mapper方法，往往只执行一次SQL，因为使用SqlSession第一次后，Mybatis
> 其会将放在缓存中，以后再查询的时候，如果没有申明需要刷新的情况下，
> SqlSession都会取出当前缓存的数据，不会再次发送SQL到数据库

* 特点
  1. 一级缓存默认开启
  2. 一级缓存作用范围为同一个sqlSession
  3. 一级缓存使用前提：缓存不能刷新

* 一级缓存的生命周期

~~~
1、一级缓存的生命周期有多长？

a、MyBatis在开启一个数据库会话时，会 创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象。Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。

b、如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用。

c、如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用。

d、SqlSession中执行了任何一个update操作(update()、delete()、insert()) ，都会清空PerpetualCache对象的数据，但是该对象可以继续使用
~~~



## 5.2 二级缓存

> 二级缓存是appliaction级别的缓存/sqlSessionFactory级别的缓存
> mybatis的二级缓存是默认关闭的

* 配置二级缓存

  1. 实体类是吸纳序列化接口

     > 补：二级缓存的数据会存储在硬盘上

  2. 使用缓存的mapper映射文件中添加配置

~~~xml
<!--开启本mapper的namespace下的二级缓存-->
<!--
	eviction:代表的是缓存回收策略，目前MyBatis提供以下策略。
        (1) LRU,最近最少使用的，一处最长时间不用的对象
        (2) FIFO,先进先出，按对象进入缓存的顺序来移除他们
        (3) SOFT,软引用，移除基于垃圾回收器状态和软引用规则的对象
        (4) WEAK,弱引用，更积极的移除基于垃圾收集器状态和弱引用规则的对象。
			这里采用的是LRU，移除最长时间不用的对形象

	flushInterval:刷新间隔时间，单位为毫秒，这里配置的是100秒刷新，
		如果你不配置它，那么当SQL被执行的时候才会去刷新缓存。

    size:引用数目，一个正整数，代表缓存最多可以存储多少个对象，
		不宜设置过大。设置过大会导致内存溢出。这里配置的是1024个对象

    readOnly:只读，意味着缓存数据只能读取而不能修改，这样设置的好处
		是我们可以快速读取缓存，缺点是我们没有办法修改缓存，
		他的默认值是false，不允许我们修改
-->
<cache eviction="LRU" flushInterval="100000" readOnly="true" size="1024"/>
~~~

* 在使用缓存的mapper映射文件的sql标签中添加配置：(默认是true：可以不用配置)

~~~xml
<!--  List<Student>  getAll1();-->
<!--可以通过设置useCache来规定这个sql是否开启缓存，ture是开启，false是关闭-->
<select id="getAll1"  resultType="Student" useCache="true">
    select * from student
</select>
<!--可以通过设置useCache来规定这个sql是否开启缓存，ture是开启，false是关闭-->
<select id="getAll2"  resultType="Student" useCache="false">
    select * from student
</select>
~~~

* 在核心配置文件中添加配置

~~~xml
 <settings>
       ....
       <!--这个配置使全局的映射器(二级缓存)启用或禁用缓存-->
       <setting name="cacheEnabled" value="true" />
   </settings>
~~~



# 6 模拟mybatis

> 默认原始dao方式
>
> 通过sqlsession的selectOne、selectList、delete、insert、update等方法 关联sql标签的id和参数来执行sql语句

> 涉及的技术：xml+反射





# 补充

##  web：三层架构

~~~
对javaweb项目结构的按业务应用进行划分：
表示层:ui：user interface
      用于显示数据和接收用户输入的数据(jsp+servlet)
业务逻辑层:BLL:Business Logic Layer
      负责关键业务的处理和数据的传递(service)
持久层:DAL:data access layer
      主要负责对数据库的直接访问(dao+entity)
~~~

## mvc：Model View Controller

~~~
M是指业务模型model，V是指用户界面view，C则是控制器controller，使用MVC的目的是将M和V的实现代码分离，从而使同一个程序可以使用不同的表现形式(pc+手机+ipad)
model:(实体类+dao+业务)   对数据和功能封装的实体类+业务类+dao类
view:(HTML+CSS+JS+jsp)  页面相关的技术(获取客户的数据+结果数据的展示)
controller:(servlet)    选择页面跳转
mvc是一种通用的软件设计模式
~~~

* 优点

~~~
1 降低耦合度：视图层和业务层分离
2 提高代码的复用性：
3 便于分工开发
4 维护性高
~~~

* 缺点

~~~
1 调试困难（单元测试困难）
2 增加了程序的复杂度
3 不适合中小型项目
~~~

## orm：对象关系映射

~~~
orm:object relational mapping 对象关系映射
	java中对象的属性与sql表中列的对应关系
解决的问题：给站位符赋值和解析结果集
~~~

## 数据库设计三大范式

* 数据库设计时 需要遵守的基本规则

### 第一范式1NF：所有列保证列的原子性

> 列数据的不可分割

内容：避免将一条数据的多段信息编写为一列，否则在使用时仍然需要对列进行拆解，
				不符合设计的理念，保证每一列的信息和数据在使用时不被拆开使用



### 第二范式2NF：消除传递依赖

> 消除传递依赖，一个表只描述一个事物；（就像java设计模式的单一职责原则）

内容：要求数据库表中的每一列都和注解相关，而不能只与主键的某一部分相关(主要针对联合主键而言)。<font color=red>也就是说在一个数据库表中，一个表中只能保存一种数据，不可以把多种数据保存在同一张数据库表中</font>



### 第三范式3NF：消除部分依赖

> 消除部分依赖：只针对于 联合主键

内容：要求数据表中的每一列数据都和主键直接相关，而不能间接相关。
				若间接相关则另外建表，用外键引用

* 错误案例:订单详情表

~~~sql
create table order_detail(
       order_id int comment  "订单编号",
       product_name varchar(11) comment "商品名字",
       product_number int comment "商品数量",
       product_price decimal comment "商品单价",
       discount  float comment "商品折扣",
       money decimal comment "总金额",
       constraint kf_2 foreign key (order_id) references order(order_id),
       primary key(order_id,product_name)
      
);
# product_price和discount 只依赖联合主键的product_name部分：
# 再定义一个商品表
~~~

* 正确案例：把部分依赖product_name的信息写道商品表中

~~~sql
create table order_detail(
       order_id int comment  "订单编号",
       product_id int comment  "商品编号",
       product_number int comment "商品数量",
       money decimal comment "总金额",
       constraint kf_21 foreign key (order_id) references order(order_id),
       constraint kf_22 foreign key (product_id) references product(product_id),
       primary key(order_id,product_id)  
);
create table product(
       product_id int primary key comment "商品编号",
       product_name varchar(11) comment "商品名字",
       product_price decimal comment "商品单价",
       discount  float comment "商品折扣"
);
~~~



## lombok

~~~
作用：可以使用简单的注解代替繁琐的生成代码，使代码变的简洁，
		但是可能会降低代码的可读性，以及泛用性，
		因为使用时需要引入响应jar包，和一定的配置
~~~

### 使用步骤

* 步骤1：项目中导入jar包

~~~
lombok-1.18.20.jar
~~~

* 步骤2：eclipse中安装lombok的插件 [参考网址](https://www.cnblogs.com/boonya/p/10691466.html)
  * 下载lombok.jar
  * 点击jar，选择安装到eclipse的安装目录下
* 步骤3：重启eclipse
* 步骤4：使用注解

~~~java
@Getter				// 生成所有属性的getter方法
@Setter				// 生成所有属性的setter方法
@ToString			// 重写toString方法
@EqualsAndHashCode	// 重写equals和hashcode方法
@NoArgsConstructor	// 无参构造方法
@AllArgsConstructor	// 全参构造方法
@Data	
//等于下面所有注解的结合体
// @ToString @EqualsAndHashCode @Getter @Setter
// @RequiredArgsConstructo
~~~
