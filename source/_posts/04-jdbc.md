---
title: JDBC
date: 2021-10-30 21:53:26
tags:
---


# 概念

> jdbc：java database connection
> 		通过java程序来连接数据库
> 		sun公司定义的通过java连接所有数据库的一组接口(标准)
> java驱动：不同数据库产品 底层的实现原理/操作不同
> 		不同数据库的产商 为自己的数据库提供的jdbc的实现类	

<!-- more -->

## 使用步骤

- 通过java来链接数据库：jdbc

> 1 导入驱动：把jdbc的实现类jar包导入到当前姓名中
> 把第三方jar包在web项目中引用
> 	把mysql-cnnertor-java-5.1.15-bin.jar
> 	复制到WebContent/WEB-INF/lib下
> 	选中jar:点击右键，选择build path
> 把第三方jar包在java项目中引用
> 	把mysql-connector-java-5.1.15-bin.jar复制项目名称下
> 	选中jar:点击右键：选择build path

~~~java
//2 准备四大参数（连接数据库的四大参数）
	url="jdbc:mysql://localhost:3306/数据库名";
	userName="";
	password="";
	diverName="com.mysql.jdbc.cj.Driver";
//3 注册驱动
	Class.forName();
//4 获取连接
	Connection con = 
    DriverManager.getConnection(url,uerNmae,password);
//5 准备sql语句
	String sql="";
//6 获取sql语句发送器对象：Statement
	Statement st = con.createStatement();
//7 通过slq语句发送器对象executeXxx方法把sql语句发送给数据库 获取影响的行数
	执行sql语句的方法分为两种，
        1 种为数据查询，从数据库中获取一定数目的记录
        2 种为更改查询，对数据库中的数据或表结构进行修改
    //对于情况1，使用 executeQuery() 方法执行;
	ResultSet set = con.executeQuery();//使用ResultSet存储结果集
	/*
		ResultSet:结果集
		方法：
		1 boolean next();	判断光标是否还有行可以遍历
		2 xxx getXxx(String columnName);根据列名获取值
		  xxx getXxx(int colunmIndex);	根据列索引获取值
		3 void close(); 关闭结果集
	*/
	List<Student> list = new ArrayList<Student>();
	while(set.next()){
        Student stu = new Student();
        stu.set属性名(set.get属性类型("列名"));
        ...
        list.add(stu);
    }
	
	//对于情况2，使用 executeUpdate() 方法执行;
	int lines = st.executeUpdate(sql);

//8 关闭资源和链接
	st.close();
	con.close();
~~~



# jdbc实现crud

~~~
数据封装：实体类---对应表
描述业务：业务类---业务逻辑
指定方面的功能封装：工具类---静态方法
~~~

## 创建实体类：根据表创建

~~~
类		 --每个单词首字母大写
属性		--除了第一个单词，其他单词首字母大写
表名/列名  --单词之间用下划线分割

实体类要求：根据表创建
1 实现序列化接口 Serializable
2 属性私有化封装
3 重写toSting
4 提供必要的constructor 构造方法
5 根据要求重写其他方法--equals方法/hashCode方法

注意：一般属性都用包装类类型
~~~

## 创建jdbc工具类

~~~java
//连接数据的步骤中，前几部为固定写法，可以将其封装为方法，方便复用。
//2 准备数据库连接参数
String drivername = "com.mysql.cj.jdbc.Driver";//8.0的要加cj
String username = "";
String password = "";
String url = "jdbc:mysql://localhost:3306/数据库名";
//3 注册数据库驱动
class.forName(diverName);
//4 获取数据库连接对象
Connection con = 
    DiverManager.getConnection(url,username,password);
//5 关闭资源和连接创建实体类封装数据
~~~

# 预处理/预编译

## preparedstatement

~~~
preparedstatement:
	是statement的子接口，把sql语句发送给数据库
	通过execute方法执行sql语句 获取结果集/影响的行数
~~~

## 通过preparedstatement实现crud

~~~java
使用的步骤和使用statement时基本相同，大致区别如下
//1 准备sql语句
	String sql = "select * from student where 列名=？";
//2 创建preparedstatement语句发送器
	Preparestatement pst = con.preparedstatement(sql);
	/*传入sql语句的模板，为？占位符设置值
		setXxx(索引,参数值)方法使用：
		1 索引是指站位符的下标，从1开始计数
		2 参数值可以为常量，也可以为对应类型变量
	*/
	pst.setInt(索引,数值);
	pst.setString(索引,数值);

//3 使用方法执行sql语句
	pst.executeUpdate();//不需要传入sql作为参数
	pst.executeQuery();

~~~

## preparedstatement的优点

~~~java
/*	preparedstatemenet预编译对象和statement相比的优点
	pre是sat的子接口：
    	都是slq语句发送给数据库通过executeXxx方法来执行sql语句*/
    
//优点1：可以防止sql攻击
	statement在执行execute方法时，把sql片段和数据通过字符串拼接后形
			成一个sql语句发送给数据，无法区分数据中伪装的sql片段。
	pre在获取预编译对象时，先发送sql模板给数据库人后再通过setXxx方法
		给站位符赋值，发送的是数据中伪装的sql片段无法被解析。
	数据库可以很容易区分，第一次发送的是sql片段，第二次发送的是数据中伪
		装的sql片段无法被解析
		
//优点2：效率高
	pre 在多次操作时(批处理)，在获取preparedstatement对象时，数据库
	只需要	对sql模板进行语法，和解析一次即可，每次操作只需要给数据库发
	送不同的占位符即可
    
    sat 每次在执行execute方法时，发送的是整个sql语句，每次都需要对sql
    语句进行语法检查和解析

//优点3：不需要sql语句的拼接
~~~

# 连接池

~~~java
数据库连接池的基本原理就是为数据库建立一个"缓冲区";

就比如流中的高效流，就是使用了"缓冲区"这一概念。
1 当程序没有访问数据库请求时，池中会存在几个空闲的数据库连接实例，
2 当程序请求访问数据库时，会将池中空闲的连接分配出去，
3 当池中的连接分配完后依然不够，池会自动创建一定数量的连接，
4 当达到池所能创建的极限后，不会继续创建新的连接
5 当池中的连接空闲一段时间后，池子会回收该连接，位置数量在一个数值。

连接池的作用：
负责创建、分配、管理、维护和释放数据库连接，它允许程序重复使用同一
个现有的数据库连接，大大缩短了运行时间，提高了执行效率。
~~~

## 常见的数据库连接池

* c3p0
* dbcp
* druid:德鲁伊？

## c3p0使用

* [使用详情参考](https://www.cnblogs.com/keystone/p/12740309.html)
* 1 导入c3p0依赖的jar
* 2 创建c3p0的配置文件：文件名字：c3p0-config.xml 文件位置：src

~~~xml
<? xml version="1.0" encoding="UTF-8"?><!-- 声明区：当前xml页面属性 -->
<c3p0-config>
  <default-config>
	<!--mysql数据库链接的各项参数-->
	<property name="driverClass">com.mysql.cj.jdbc.Driver</property>
	<property name="jdbcUrl">jdbc:mysql://localhost:3306/数据库名</property>
	<property name="user">root</property>
	<property name="password">123</property>
	
	<!--配置数据库连接池的初始连接数、最小连接数、获取连接数、最大连接数、最大空闲时间-->
    <property name="initialPoolSize">10</property>初始连接数
    <property name="minPoolSize">10</property>最小连接数
	<property name="acquireIncrement">100</property>获取连接数
	<property name="maxPoolSize">100</property>最大连接数
	<property name="maxIdleTime">30</property>最大空闲时间
  </default-config>
</c3p0-config>
~~~

* 3 创建c3p0的工具类

~~~java
//静态属性获取连接池对象
private satic DataSource ds = new CombPooledDataSource();
//获取连接
ds.getConnection();//可能会出异常，需要处理
//释放连接:通过DataSource获取的Connection对象的close方法已经被重写；
//			不再是关闭连接，而是把连接归还给连接池
public static void release(ResultSet set,Statement sta,Connection con){
	set.close;//可能会抛出异常
	sta.close;
	con.close;
}
~~~



