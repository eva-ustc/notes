# Mybatis面试

## 1. Mybatis的架构

```
	1.SqlSession：作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能；
	2.Executor：MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护；
	3.StatementHandler：封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集合。
	4.ParameterHandler：负责对用户传递的参数转换成JDBC Statement 所需要的参数；
	5.ResultSetHandler：负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；
	6.TypeHandler：负责java数据类型和jdbc数据类型之间的映射和转换；
	7.MappedStatement：MappedStatement维护了一条<select|update|delete|insert>节点的封装；
	8.SqlSource：负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回；
	9.BoundSql：表示动态生成的SQL语句以及相应的参数信息；
	10.Configuration：MyBatis所有的配置信息都维持在Configuration对象之中；
```

![](https://upload-images.jianshu.io/upload_images/2062729-a2f20529d6d908a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/905/format/webp)

首先加载配置文件，mybatis有两个配置文件：

       **SqlMapConfig.xml**，此文件作为mybatis的**全局配置文件**，配置了mybatis的运行环境等信息。

       **mapper.xml**文件即**sql映射文件**，文件中配置了操作数据库的sql语句。此文件需要在SqlMapConfig.xml中加载。

（1）通过mybatis环境等配置信息构造**SqlSessionFactory**即会话工厂

（2）由会话工厂创建**sqlSession**即会话，与数据库的一次连接,操作数据库需要通过sqlSession进行。

（3）mybatis底层自定义了**Executor执行器**接口操作数据库，Executor接口有两个实现，一个是基本执行器、一个是缓存执行器。

（4）**MappedStatement**也是mybatis一个底层封装对象，它包装了mybatis配置信息及sql映射信息等。mapper.xml文件中一个sql对应一个Mapped Statement对象，sql的id即是Mapped statement的id。

（5）**MappedStatement对sql执行输入参数进行定义**，包括HashMap、基本类型、pojo，Executor通过MappedStatement在执行sql前将输入的java对象映射至sql中，输入参数映射就是jdbc编程中对preparedStatement设置参数。

（6）**MappedStatement对sql执行输出结果进行定义**，包括HashMap、基本类型、pojo，Executor通过Mapped Statement在执行sql后将输出结果映射至java对象中，输出结果映射过程相当于jdbc编程中对结果的解析处理过程。

```java
	// 读取配置文件
	InputStream inputStream = Resources.getResourceAsStream(resource);
	// 根据配置文件获得SQLSessionFactory
     SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
     // 由SQLSessionFactory创建SQLSession
     SqlSession session = factory.openSession(true);
     // 直接由SQLsession调用方法,输入参数执行
     User user = session.selectOne("ustc.sse.xmlmapper.UserMapper.getUserById", 1);
 or	 // 或者传入Mapper接口获得对应的Mapper代理对象
     UserMapper mapper = session.getMapper(UserMapper.class);
	// 利用Mapper代理对象去执行sql语句
     User user = mapper.getUserById("1079961649097609216");
```



## 2. mybatis和Hibernate的区别？

答：Mybatis和Hibernate框架都是持久层框架，底层封装的都是jdbc。它们的区别是：

       Mybatis是一个不完全的ORM框架，需要程序员自己编写Sql语句；Mybatis无法做到数据库无关性，适合对关系数据模型要求不高的软件开发。

       Hibernate是一个完全的ORM框架，不需要程序员自己编写Sql语句；Hibernate数据库无关性好，适合于对关系数据模型要求高的软件开发。

       Mybatis和Hibernate框架各自都有非常广泛的应用领域，本身并无好坏之分，适合就是最好的。 



## 3. **当实体类中的属性名和表中的字段名不一样，怎么办？**

**答：**两种方法。

​        1.通过在查询的sql语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。

​        2.通过<resultMap>来映射字段名和实体类属性名的一一对应的关系（推荐使用）



## 4. Mapper映射文件或Mapper接口

​	这里的Dao接口实际上指的是Mapper接口，Mapper接口和Mapper.xml文件是对应的。

​	接口的全限名，就是映射文件中的**namespace**的值，接口的方法名，就是映射文件中MappedStatement的id值，接口方法内的参数，就是传递给sql的参数。**Mapper接口是没有实现类的**，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MappedStatement。

       **Dao接口里的方法，是不能重载的**，因为是全限名+方法名的保存和寻找策略

       Dao接口的工作原理是**JDK动态代理**，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，**转而执行MappedStatement所代表的sql**，然后将sql执行结果返回。



## 5. Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？

​	答：第一种是使用<resultMap>标签，**逐一定义列名和对象属性名之间的映射关系**，**第二种是使用sql列的别名功能，将列别名书写为对象属性名。**

​	有了列名和属性名的映射关系后，Mybatis通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

## 6. sqlMapConfig.xml的配置一般包含哪些内容呢？

答：SqlMapConfig.xml中配置的内容和顺序如下：（顺序不能颠倒）

               

```
  	properties（属性）
	settings（全局配置参数）
	typeAliases（类型别名）
	typeHandlers（类型处理器）
	objectFactory（对象工厂）
	plugins（插件）
	environments（环境集合属性对象）
		environment（环境子属性对象）
			transactionManager（事务管理）
			dataSource（数据源）
	mappers（映射器）
```



## 7. 简单的说一下MyBatis的一级缓存和二级缓存？

​	Mybatis首先去缓存中查询结果集，如果没有则查询数据库，如果有则从缓存取出返回结果集就不走数据库。Mybatis内部存储缓存使用一个**HashMap**，key为**hashCode+sqlId+Sql**语句。value为从查询出来映射生成的java对象,一级缓存是Session域的,即会话级别,会话关闭或提交会将缓存写入二级缓存;

​	Mybatis的二级缓存即查询缓存，它的作用域是一个mapper的**namespace**，即在同一个namespace中查询sql可以从缓存中获取数据。二级缓存是可以跨SqlSession的。



### 8. $和#的区别

​	动态 SQL 是 mybatis 的强大特性之一，也是它优于其他 ORM 框架的一个重要原因。mybatis 在对 sql 语句进行预编译之前，会对 sql 进行动态解析，解析为一个 BoundSql 对象，也是在此处对动态 SQL 进行处理的。在动态 SQL 解析阶段， #{ } 和 ${ } 会有不同的表现。

\#{ }：解析为一个 JDBC 预编译语句（prepared statement）的参数标记符。
例如，Mapper.xml中如下的 sql 语句：

```
select * from user where name = #{name};
```

 动态解析为：

```
select * from user where name = ?; 
```

​	一个 #{ } 被解析为一个参数占位符 ? 。
​	而${ } 仅仅为一个纯碎的 string 替换，在动态 SQL 解析阶段将会进行变量替换。

例如，Mapper.xml中如下的 sql：

```
select * from user where name = ${name};
```

​	 当我们传递的参数为 "Jack" 时，上述 sql 的解析为：

```
select * from user where name = "Jack";
```

​	预编译之前的 SQL 语句已经不包含变量了，完全已经是常量数据了。 

​	综上所得， **${ } 变量的替换阶段是在动态 SQL 解析阶段，而 #{ }变量的替换是在 DBMS 中。**













