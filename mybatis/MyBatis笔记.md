视频地址：https://www.bilibili.com/video/BV1VP4y1c7j7?from=search&seid=1895879482480987332&spm_id_from=333.337.0.0

# Mybatis简介

## MyBatis历史
-    MyBatis最初是Apache的一个开源项目iBatis, 2010年6月这个项目由Apache Software Foundation迁移到了Google Code。随着开发团队转投Google Code旗下，iBatis3.x正式更名为MyBatis。代码于2013年11月迁移到Github
- iBatis一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。iBatis提供的持久层框架包括SQL Maps和Data Access Objects（DAO）
## MyBatis特性
1. MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架
2. MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集
3. MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录
4. MyBatis 是一个 半自动的ORM（Object Relation Mapping）框架
## MyBatis下载
- [MyBatis下载地址](https://github.com/mybatis/mybatis-3)
- ![](https://image.imxyu.cn/file/MyBatis%E4%B8%8B%E8%BD%BD.png)
## 和其它持久化层技术对比
- JDBC  
	- SQL 夹杂在Java代码中耦合度高，导致硬编码内伤  
	- 维护不易且实际开发需求中 SQL 有变化，频繁修改的情况多见  
	- 代码冗长，开发效率低

> jdbc代码都是在java文件中写的，编译出来的都是class文件，项目运行期间如果想改的话没法改

- Hibernate 和 JPA
	- 操作简便，开发效率高  
	- 程序中的长难复杂 SQL 需要绕过框架  
	- 内部自动生产的 SQL，不容易做特殊优化  
	- 基于全映射的全自动框架，大量字段的 POJO 进行部分映射时比较困难。  
	- 反射操作太多，导致数据库性能下降
- MyBatis
	- 轻量级，性能出色  
	- SQL 和 Java 编码分开，功能边界清晰。Java代码专注业务、SQL语句专注数据  
	- 开发效率稍逊于HIbernate，但是完全能够接受
# 搭建MyBatis
## 开发环境
- IDE：idea 2019.2  
- 构建工具：maven 3.5.4  
- MySQL版本：MySQL 5.7  
- MyBatis版本：MyBatis 3.5.7
## 创建maven工程
- 打包方式：jar
- 引入依赖

~~~xml

<dependencies>
	<!-- Mybatis核心 -->
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.5.7</version>
	</dependency>
	<!-- junit测试 -->
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.12</version>
		<scope>test</scope>
	</dependency>
	<!-- MySQL驱动 -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>5.1.3</version>
		</dependency>
</dependencies>

~~~
## 创建MyBatis的核心配置文件
>习惯上命名为`mybatis-config.xml`，这个文件名仅仅只是建议，并非强制要求。将来整合Spring之后，这个配置文件可以省略，因为到时候这个配置文件会全部交给spring管理，所以大家操作时可以直接复制、粘贴。
>核心配置文件主要用于配置连接数据库的环境以及MyBatis的全局配置信息
>核心配置文件存放的位置是src/main/resources目录下
```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration  
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
"http://mybatis.org/dtd/mybatis-3-config.dtd">  
<configuration>  
	<!--设置连接数据库的环境-->  
	<environments default="development">  
		<environment id="development">  
            <!--当前事务处理方式以jdbc的方式来管理，事务的提交和回滚都由jdbc 一样手动管理-->  
			<transactionManager type="JDBC"/>  
			<dataSource type="POOLED">  
				<property name="driver" value="com.mysql.cj.jdbc.Driver"/>  
				<property name="url" value="jdbc:mysql://localhost:3306/MyBatis"/>  
				<property name="username" value="root"/>  
				<property name="password" value="123456"/>  
			</dataSource>  
		</environment>  
	</environments>  
	<!--引入映射文件-->  
	<mappers>  
		<mapper resource="mappers/UserMapper.xml"/>  
	</mappers>  
</configuration>
```
## 创建mapper接口
>MyBatis中的mapper接口相当于以前的dao。但是区别在于，mapper仅仅是接口，我们不需要提供实现类
```java
package com.atguigu.mybatis.mapper;  
  
public interface UserMapper {  
	/**  
	* 添加用户信息  
	*/  
	int insertUser();  
}
```
## 创建MyBatis的映射文件
- 相关概念：ORM（Object Relationship Mapping）对象关系映射。  
	- 对象：Java的实体类对象  
	- 关系：关系型数据库  
	- 映射：二者之间的对应关系

| Java概念 | 数据库概念 |
| --- | --- |
| 类 | 表 |
| 属性 | 字段/列 |
| 对象 | 记录/行 |

- 映射文件的命名规则
	- 表所对应的实体类的类名+Mapper.xml
	- 例如：表t_user，映射的实体类为User，所对应的映射文件为UserMapper.xml 
	- 因此一个映射文件对应一个实体类，对应一张表的操作
	- MyBatis映射文件用于编写SQL，访问以及操作表中的数据
	- MyBatis映射文件存放的位置建议在src/main/resources/mappers目录下

> 相应的我们要在mybatis-config.xml 的这个主配置文件中，引入mapper文件的路径。（这个文件不用记，后续整合spring后记不用了）

```xml
<mappers> 
<mapper resource="mappers/UserMapper.xml"/> 
</mappers>
```

> 需要注意的是因为mappers 相关的xml文件都是放在**resource** 路径下的，不是java src路径，代表的只是目录，不能用 mapper. 这种点的方式，需要以文件目录/ 斜杠的形式



```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.atguigu.mybatis.mapper.UserMapper">  
	<!--int insertUser();-->  
	<insert id="insertUser">  
		insert into t_user values(null,'张三','123',23,'女')  
	</insert>  
</mapper>
```
> MyBatis中可以面向接口操作数据，要保证两个一致
>
> 1、mapper接口的全类名和映射文件的命名空间（namespace）保持一致（这样通过接口就能找到相应的xml文件）
>
> 2、mapper接口中方法的方法名和映射文件中编写SQL的标签的id属性保持一致（这样就能通过接口中的方法找到xml文件中的方法）



## 通过junit测试功能

- SqlSession：代表Java程序和数据库之间的会话。（HttpSession是Java程序和浏览器之间的会话）
- SqlSessionFactory：是“生产”SqlSession的“工厂”
- 工厂模式：如果创建某一个对象，使用的过程基本固定，那么我们就可以把创建这个对象的相关代码封装到一个“工厂类”中，以后都使用这个工厂类来“生产”我们需要的对象
```java
public class UserMapperTest {
    @Test
    public void testInsertUser() throws IOException {
        //读取MyBatis的核心配置文件
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        //获取SqlSessionFactoryBuilder对象
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
        SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
        //获取sqlSession，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
        //SqlSession sqlSession = sqlSessionFactory.openSession();
	    //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交  
		SqlSession sqlSession = sqlSessionFactory.openSession(true);
        //通过代理模式创建UserMapper接口的代理实现类对象
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        //调用UserMapper接口中的方法，就可以根据UserMapper的全类名匹配元素文件，通过调用的方法名匹配映射文件中的SQL标签，并执行标签中的SQL语句
        int result = userMapper.insertUser();
        //提交事务
        //sqlSession.commit(); 如果没有设置true则需要手动提交 sqlSessionFactory.openSession(); 
        System.out.println("result:" + result);
    }
}
```
- 此时需要手动提交事务，如果要自动提交事务，则在获取sqlSession对象时，使用`SqlSession sqlSession = sqlSessionFactory.openSession(true);`，传入一个Boolean类型的参数，值为true，这样就可以自动提交

> 主要就是通过 sqlSession.getMapper(UserMapper.class); 利用代理模式通过我们传入的接口类型在找到对应的xml文件（因为我们在xml文件中写了namespace对应mapper），然后给我们创建出接口对应的实现类，再根据我们接口中调用的方法，执行xml中我们定义的方法（这个是根据我们在xml中定义的id就能找到对应的方法）
>
> 因此编写xml文件的时候，保持**两个一致原则**很重要  1、namespace对应mapper接口  2、 id对应方法

## 加入log4j日志功能
1. 加入依赖
	```xml
	<!-- log4j日志 -->
	<dependency>
	<groupId>log4j</groupId>
	<artifactId>log4j</artifactId>
	<version>1.2.17</version>
	</dependency>
	```
2. 加入log4j的配置文件
	- log4j的配置文件名为**log4j.xml**，存放的位置是src/main/resources目录下
	- 日志的级别：FATAL(致命)>ERROR(错误)>WARN(警告)>INFO(信息)>DEBUG(调试) 从左到右打印的内容越来越详细
	
	> 注意所以日志文件的文件名一般都是固定的名字
	
	```xml
	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
	<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
	    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
	        <param name="Encoding" value="UTF-8" />
	        <layout class="org.apache.log4j.PatternLayout">
				<param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS} %m (%F:%L) \n" />
	        </layout>
	    </appender>
	    <!--这个是java中的sql日志级别-->
	    <logger name="java.sql">
	        <level value="debug" />
	    </logger>
	    <!--这个是mybatis的日志级别-->
	    <logger name="org.apache.ibatis">
	        <level value="info" />
	    </logger>
	    <!--这个是我们当前的日志级别是debug，因此java中的和ibatis的信息都会打印出来，并且以控制台的方式输出-->
	    <root>
	        <level value="debug" />
	        <appender-ref ref="STDOUT" />
	    </root>
	</log4j:configuration>
	```

加入日志后的运行结果：

第一个行是mybatis的sql语句的执行，第二行是java在接口中传入的参数信息 ，我们这里没有传入参数，是写死的。所以为空

第三行是我们执行得到的结果，影响的行数 1行

![image-20220310103650664](https://image.imxyu.cn/file/image-20220310103650664.png)



# 核心配置文件详解

核心配置文件简单了解一下就可以，后续会使用spring整合之后就不用了，里面的所有标签都可以在spring中配置，这里只需要能复制看懂，会改改属性就行。不用会写

>核心配置文件中的标签必须按照固定的顺序(有的标签可以不写，但顺序一定不能乱)：
properties、settings、typeAliases、typeHandlers、objectFactory、objectWrapperFactory、reflectorFactory、plugins、environments、databaseIdProvider、mappers
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//MyBatis.org//DTD Config 3.0//EN"
        "http://MyBatis.org/dtd/MyBatis-3-config.dtd">
<configuration>
    <!--引入properties文件，此时就可以${属性名}的方式访问属性值-->
    <properties resource="jdbc.properties"></properties>
    <settings>
        <!--将表中字段的下划线自动转换为驼峰-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!--开启延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>
    <typeAliases>
        <!--
        typeAlias：设置某个具体的类型的别名，这样我们在xml文件中可以直接使用别名，不需要写全类名了
        属性：
        type：需要设置别名的类型的全类名
        alias：设置此类型的别名，且别名不区分大小写。若不设置此属性，该类型拥有默认的别名，即类名
        -->
        <!--<typeAlias type="com.atguigu.mybatis.bean.User"></typeAlias>-->
        <!--<typeAlias type="com.atguigu.mybatis.bean.User" alias="user">
        </typeAlias>-->
        <!--以包为单位，以后一般都会用这种直接引入一个包的，不会用上面的那个，设置改包下所有的类型都拥有默认的别名，即类名且不区分大小写-->
        <package name="com.atguigu.mybatis.bean"/>
    </typeAliases>
    <!--
    environments：设置多个连接数据库的环境
    属性：
	    default：设置默认使用的一个环境environment的id
    -->
    <environments default="mysql_test">
        <!--
        environment：设置具体的连接数据库的环境信息
        属性：
	        id：设置环境的唯一标识，可通过environments标签中的default设置某一个环境的id，表示默认使用的环境
        -->
        <environment id="mysql_test">
            <!--
            transactionManager：设置事务管理方式
            属性：
	            type：设置事务管理方式，type="JDBC|MANAGED"
	            type="JDBC"：设置当前环境的事务管理都必须手动处理
	            type="MANAGED"：设置事务被管理，例如spring ，可以通过spring中的声明式事务进行管理
            -->
            <transactionManager type="JDBC"/>
            <!--
            dataSource：设置数据源。当使用spring之后可以使用spring中的数据源
            属性：
	            type：设置数据源的类型，type="POOLED|UNPOOLED|JNDI"
	            type="POOLED"：使用数据库连接池，即会将创建的连接进行缓存，下次使用可以从缓存中直接获取，不需要重新创建
	            type="UNPOOLED"：不使用数据库连接池，即每次使用连接都需要重新创建
	            type="JNDI"：调用上下文中的数据源
            -->
            <dataSource type="POOLED">
                <!--设置驱动类的全类名，下面的这些都是通过读取property文件中的key值，property文件可以在上面使用标签引入即可-->
                <property name="driver" value="${jdbc.driver}"/>
                <!--设置连接数据库的连接地址-->
                <property name="url" value="${jdbc.url}"/>
                <!--设置连接数据库的用户名-->
                <property name="username" value="${jdbc.username}"/>
                <!--设置连接数据库的密码-->
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <!--引入映射文件-->
    <mappers>
        <!-- <mapper resource="UserMapper.xml"/> -->
        <!--
        以包为单位，将包下所有的映射文件引入核心配置文件，以后都会用这种，因为mapper 文件会很多
        以包为单位必须注意的2点：
			1. 此方式必须保证mapper接口和mapper映射文件必须在相同的包下
			2. mapper接口要和mapper映射文件的名字一致
        -->
        <package name="com.atguigu.mybatis.mapper"/>
    </mappers>
</configuration>
```


使用<package name="com.atguigu.mybatis.mapper"/> 后文件目录结构：

- ![](https://image.imxyu.cn/file/mapper%E6%8E%A5%E5%8F%A3%E5%92%8Cmapper%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6%E5%9C%A8%E5%90%8C%E4%B8%80%E5%8C%85%E4%B8%8B.png)



> 这里需要注意的是因为我们是在resouces 中创建包，而resouces 中没有创建package的选项，所以我们需要创建directory 同时以/ 斜杠作为每一层的目录，不能用 . 点，用点代表一个文件夹的名字，就不是多层的目录了

![image-20220310115820145](https://image.imxyu.cn/file/image-20220310115820145.png)

在java 源码路径下我们可以创建package ，就可以用 . 点来分隔

![image-20220310115844518](https://image.imxyu.cn/file/image-20220310115844518.png)



# 使用idea中的模板保存配置文件

每次都需要 复制文件头和里面的属性，很麻烦，我们可以利用idea提供的模板文件来保存起来。下次直接创建即可

我们可以利用idea中的模板，来保存这个配置文件的格式，下次我们可以快速使用模板来进行创建这个文件

下面是我们创建的通用的模板，package 的名字没有写，数据源的信息是从配置文件中加载的，默认是从jdbc.properties 这个文件

下面我们利用idea 来将这个模板保存下来

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <properties resource="jdbc.properties" />

    <typeAliases>
        <package name=""/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <package name=""/>
    </mappers>
</configuration>
```

找到settings中的这个，点击加号创建一个模板，这里的name 就是模板名字，注意这个不是最后的文件名。文件名可以我们创建的时候自己指定， 后缀是xml。

这样我们下次可以直接创建这个文件，直接写上想要的名字，推荐还是用mabatis-config 这个名字，见名知意， 不需要写后缀，idea会用我们这里设置的后缀。

![image-20220310141846686](https://image.imxyu.cn/file/image-20220310141846686.png)



创建好了模板之后，我们点击new 就可以快速创建这个模板文件了

![image-20220310142102824](https://image.imxyu.cn/file/image-20220310142102824.png)

然后指定文件名字

![image-20220310142130809](https://image.imxyu.cn/file/image-20220310142130809.png)

可以看到很方便

![image-20220310142147921](https://image.imxyu.cn/file/image-20220310142147921.png)





同样的我们可以创建xml的模板，这是我们最简单的xml， 其中namespace 是不确定的，所以空出来

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="">

 

</mapper>
```

![image-20220310142924826](https://image.imxyu.cn/file/image-20220310142924826.png)

下次就可以通过模板来创建这个mapper文件了

![image-20220310142951118](https://image.imxyu.cn/file/image-20220310142951118.png)



# 默认的类型别名

我们之前说过了可以以包为单位设置返回的pojo的别名，mybatis给我们提供了很多的基础类型也有默认的别名。我们可以直接使用即可

![](https://image.imxyu.cn/file/%E9%BB%98%E8%AE%A4%E7%9A%84%E7%B1%BB%E5%9E%8B%E5%88%AB%E5%90%8D1.png)
![](https://image.imxyu.cn/file/%E9%BB%98%E8%AE%A4%E7%9A%84%E7%B1%BB%E5%9E%8B%E5%88%AB%E5%90%8D2.png)

# MyBatis的增删改查
1. 添加
	```xml
	<!--int insertUser();-->
	<insert id="insertUser">
		insert into t_user values(null,'admin','123456',23,'男','12345@qq.com')
	</insert>
	```
2. 删除
	```xml
	<!--int deleteUser();-->
    <delete id="deleteUser">
        delete from t_user where id = 6
    </delete>
	```
3. 修改
	```xml
	<!--int updateUser();-->
    <update id="updateUser">
        update t_user set username = '张三' where id = 5
    </update>
	```
4. 查询一个实体类对象
	```xml
   <!--User getUserById();-->  
	<select id="getUserById" resultType="com.atguigu.mybatis.bean.User">  
		select * from t_user where id = 2  
	</select>
	```
5. 查询集合
	```xml
	<!--List<User> getUserList();-->
	<select id="getUserList" resultType="com.atguigu.mybatis.bean.User">
		select * from t_user
	</select>
	```
- 注意：

	1. 查询的标签select必须设置属性resultType或resultMap，用于设置实体类和数据库表的映射关系  
		- resultType：自动映射，用于属性名和表中字段名一致的情况  
		- resultMap：自定义映射，用于一对多或多对一或字段名和属性名不一致的情况  
	2. 当查询的数据为多条时，不能使用实体类作为返回值，只能使用集合，否则会抛出异常TooManyResultsException；但是若查询的数据只有一条，可以使用实体类或集合作为返回值
# MyBatis获取参数值的两种方式（重点）

## 原生的jdbc获取参数的方式

原生的jdbc获取参数的方式有两种，一种是字符串拼接的方式，另一种是使用占位符的方式，我们来回顾一下

```java
    @Test
    public void testJDBC() throws SQLException, ClassNotFoundException {
        String username = "cherry";
        Class.forName("");
        Connection connection = DriverManager.getConnection("", "", "");
        // 1. 字符串拼接 ->获得预编译对象 -》sql注入问题
        PreparedStatement preparedStatement = connection.prepareStatement("select * from t_user where username = '" + username + "'");

        // 2. 占位符的方式可以避免字符串拼接的繁琐，同时避免sql注入的方式
        PreparedStatement ps2 = connection.prepareStatement("select * from t_user where username = ?");
        ps2.setString(1, username);
    }

```

在mybatis中的xml文件中的sql实际还是要通过接口传来的参数来进行赋值的，而mybatis中的#{}和${}就是对上面两种方式的封装实现

- MyBatis获取参数值的两种方式：${}和#{}  
- ${}的本质就是字符串拼接，#{}的本质就是占位符赋值  
- ${}使用字符串拼接的方式拼接sql，若为字符串类型或日期类型的字段进行赋值时，需要手动加单引号；**但是#{}使用占位符赋值的方式拼接sql，此时为字符串类型或日期类型的字段进行赋值时，可以自动添加单引号**

> 因此能使用#{} 的就不使用${} ，但是有些情况只能使用${} ，下面我们来看一下



## 单个字面量类型的参数
- 若mapper接口中的方法参数为单个的字面量类型，此时可以使用\${}和#{}以任意的名称（最好见名识意）获取参数的值，对于使用#{}是使用 ？ 占位符设置的参数，而${}是使用字符串拼接方式，不是设置参数值的方式，所以注意${}需要手动加**单引号**，否则会报错

```xml
<!--User getUserByUsername(String username);-->
<select id="getUserByUsername" resultType="User">
	select * from t_user where username = #{username}
</select>
```
```xml
<!--User getUserByUsername(String username);-->
<select id="getUserByUsername" resultType="User">  
	select * from t_user where username = '${username}'  
</select>
```


## 多个字面量类型的参数

- 若mapper接口中的方法参数为多个时，此时MyBatis会自动将这些参数放在一个map集合中

	1. 以arg0,arg1...为键，以参数为值；
	2. 以param1,param2...为键，以参数为值；
- 因此只需要通过\${}和#{}访问map集合的键就可以获取相对应的值，**注意${}需要手动加单引号。**
- 使用arg或者param都行，要注意的是，arg是从arg0开始的，param是从param1开始的
```xml
<!--User checkLogin(String username,String password);-->
<select id="checkLogin" resultType="User">  
	select * from t_user where username = #{arg0} and password = #{arg1}  
</select>
```
```xml
<!--User checkLogin(String username,String password);-->
<select id="checkLogin" resultType="User">
	select * from t_user where username = '${param1}' and password = '${param2}'
</select>
```
> 注意这里我们不能使用#{username} 和#{password}来获取，会报错，当多个参数时会帮我们放到这个map中

## map集合类型的参数

- 若mapper接口中的方法需要的参数为多个时，此时可以手动创建map集合，将这些数据放在map中只需要通过\${}和#{}访问map集合的键就可以获取相对应的值，**注意${}需要手动加单引号**
```xml
<!--User checkLoginByMap(Map<String,Object> map);-->
<select id="checkLoginByMap" resultType="User">
	select * from t_user where username = #{username} and password = #{password}
</select>
```
```java
@Test
public void checkLoginByMap() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	ParameterMapper mapper = sqlSession.getMapper(ParameterMapper.class);
	Map<String,Object> map = new HashMap<>();
	map.put("usermane","admin");
	map.put("password","123456");
	User user = mapper.checkLoginByMap(map);
	System.out.println(user);
}
```
## 实体类类型的参数
- 若mapper接口中的方法参数为实体类对象时此时可以使用\${}和#{}，通过访问实体类对象中的属性名获取属性值，**注意${}需要手动加单引号**
```xml
<!--int insertUser(User user);-->
<insert id="insertUser">
	insert into t_user values(null,#{username},#{password},#{age},#{sex},#{email})
</insert>
```
```java
@Test
public void insertUser() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	ParameterMapper mapper = sqlSession.getMapper(ParameterMapper.class);
	User user = new User(null,"Tom","123456",12,"男","123@321.com");
	mapper.insertUser(user);
}
```
> 注意这里要提一下什么是属性？
>
> 属性不是成员变量，是对应的get/set 方法，实际上是通过get()来获取，set方法去赋值的 。不能说没了成员变量只有get/set方法就说没有属性了，那是不对的

## 使用@Param标识参数

之前我们说过如果有多个参数的话，会帮我们放到一个map中（对应的arg0,arg1/parm1,parm2），此时我们可以使用@Param 来指定map中的键的名字

- 可以通过@Param注解标识mapper接口中的方法参数，此时，会将这些参数放在map集合中 

	1. 以@Param注解的value属性值为键，以参数为值；
	2. 以param1,param2...为键，以参数为值；
- 只需要通过\${}和#{}访问map集合的键就可以获取相对应的值，注意${}需要手动加单引号
```xml
<!--User CheckLoginByParam(@Param("username") String username, @Param("password") String password);-->
    <select id="CheckLoginByParam" resultType="User">
        select * from t_user where username = #{username} and password = #{password}
    </select>
```
```java
@Test
public void checkLoginByParam() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	ParameterMapper mapper = sqlSession.getMapper(ParameterMapper.class);
	mapper.CheckLoginByParam("admin","123456");
}
```
> 并且注意如果使用@Parm 之后我们会使用我们自定义的名字设置map中的key的名字，map中还会保留parm1,parm2 这样以parm开头的key作为默认键的值，而对应的arg这个键名 就没有了

可以看到下面是我们使用@Parm指定完自定义的key 名字之后，随便用一个名来作为key获取值报错信息，这里会告诉我们availabe parameters为这几个。

![image-20220310150654369](https://image.imxyu.cn/file/image-20220310150654369.png)

## 总结

- 实际使用的时候不需要分这么多情况，建议分成两种情况进行处理

	1. 可以以实体类类型的默认属性名
	2. 使用@Param标识参数
对于集合和数组类型的参数，我们同样可以以@param 自定义key 来获取，下面讲动态sql的时候会具体讲到

## @Parm源码分析

1）打断点，进入debug模式

![在这里插入图片描述](https://image.imxyu.cn/file/ab09321267f84432968359af620f5522.png)

选择step into进行调试

![在这里插入图片描述](https://image.imxyu.cn/file/fdb54fa3a33941e2bc7c88e8e47d88e1.png)


（2）Mapper底层使用代理模式创建，我们按step into进入缓存cachedInvoker的invoke()方法

![在这里插入图片描述](https://image.imxyu.cn/file/86fd95650f164574b9bc435722f50e53.png)

（3）进入了invoke()，再进一步进入了mapperMethod的execute()方法（step into）。

![在这里插入图片描述](https://image.imxyu.cn/file/b6426b790e484cfc86096e0b79461e73.png)

`execute()`方法中包含一个switch语句，分别对应增删改查等关键字对应的处理流程。

![在这里插入图片描述](https://image.imxyu.cn/file/b615d583767a4990a5853e427cdf47c9.png)

（4）点击step over跳进switch语句内部，光标放在command上，点击加号，查看command内容。name为该sql语句的唯一标识，type为对应操作，即SELECT。

![在这里插入图片描述](https://image.imxyu.cn/file/11e90a04a234414da14c3fedada3af2f.png)

![在这里插入图片描述](https://image.imxyu.cn/file/e33f0139e6b14ad0a4d14f58fbe38203.png)

再点击step into，发现`convertArgsToSqlCommandParam()`方法是由`paramNameResolver`参数名称解析器中的`getNamedParams()`获取命名参数方法解析的，进一步step into进这个方法。

![在这里插入图片描述](https://image.imxyu.cn/file/376f8214fb7d42468345e04f93f7e979.png)

5）进入`getNamedParams()`方法，查看names，发现names是一个map集合，包含了传进来的`"username"、"password"` 参数。

![在这里插入图片描述](https://image.imxyu.cn/file/c2ec5374002041089625605b0d63feca.png)

我们发现这个不就是我们@Parm注解中自定义的key的值吗，我们看一下这个names 是从哪来的，由结果再翻上去看`@Param`的解析过程。

![在这里插入图片描述](https://image.imxyu.cn/file/7167d31acfc04f73b0e88beb2cab632d.png)

这个解析完了得到的就是这样一个Map:{0,"username" } {1，"passwrord"} ，其中键对应的是索引下标

然后将这个map 转换成了unmodifableSotedMap , 注意此时map 中的顺序是固定的，不会混乱，方便后面根据参数的索引一 一对应

（6）回到`getNamedParams()`方法，继续step over

![在这里插入图片描述](https://image.imxyu.cn/file/1b4ccfb15a584fa8953d5e66bb9cbae3.png)

这里首先会判断这个属性hasParamAnnotation ， 这个我们上面如果有@Parm注解就已经把这个属性设置为true 了

然后for循环这个map , args [ ]是我们接口上传进来的参数值[admin,123]，将这个参数值和上面解析的names 这个map 一 一对应，就保存到了我们的map中

于是就有了{username,admin} {password,123} 这样的map 

下面会判断，如果我们的names 不包含param1,param2 这样的value ，也会以param1 ,param2 作为key 以args[] 对应的value 放入到map 中

> 因此我们标有@Param注解中肯定会有param1， param2 这样的key

# MyBatis的各种查询功能

**之前我们说过mybatis的查询在xml中必须要使用@resultType或者@resultMap 设置**

下面我们要说的是不同情况的返回值类型在**接口中**需要通过什么样的类型来接收

1. 如果查询出的数据只有一条，可以通过
	1. 实体类对象接收
	2. List集合接收
	3. Map集合接收，结果`{password=123456, sex=男, id=1, age=23, username=admin}`
2. 如果查询出的数据有多条，一定不能用实体类对象接收，会抛异常TooManyResultsException，可以通过
	1. 实体类类型的LIst集合接收
	2. Map类型的LIst集合接收
	3. 在mapper接口的方法上添加@MapKey注解
## 查询一个实体类对象
```java
/**
 * 根据用户id查询用户信息
 * @param id
 * @return
 */
User getUserById(@Param("id") int id);
```
```xml
<!--User getUserById(@Param("id") int id);-->
<select id="getUserById" resultType="User">
	select * from t_user where id = #{id}
</select>
```
## 查询一个List集合
```java
/**
 * 查询所有用户信息
 * @return
 */
List<User> getUserList();
```
```xml
<!--List<User> getUserList();-->
<select id="getUserList" resultType="User">
	select * from t_user
</select>
```
## 查询单个数据
```java
/**  
 * 查询用户的总记录数  
 * @return  
 * 在MyBatis中，对于Java中常用的类型都设置了类型别名  
 * 例如：java.lang.Integer-->int|integer  
 * 例如：int-->_int|_integer  
 * 例如：Map-->map,List-->list  
 */  
int getCount();
```
```xml
<!--int getCount();-->
<select id="getCount" resultType="_integer">
	select count(id) from t_user
</select>
```
## 查询一条数据为map集合
```java
/**  
 * 根据用户id查询用户信息为map集合  
 * @param id  
 * @return  
 */  
Map<String, Object> getUserToMap(@Param("id") int id);
```
```xml
<!--Map<String, Object> getUserToMap(@Param("id") int id);-->
<select id="getUserToMap" resultType="map">
	select * from t_user where id = #{id}
</select>
<!--结果：{password=123456, sex=男, id=1, age=23, username=admin}-->
```
> 我们上面说过了基本数据类型都有别名，所以这里我们可以直接使用  resultType="map"，不需要指定具体的包名
>
> 对于map 集合的查询我们以后还是会经常使用的，因为从数据库查询出来的属性可能不会对应任意一个pojo 对象，此时我们就要使用map来保存，比如我们的多表查询，返回的不止一个表中的字段值，我们就可以通过map来保存，然后转换成json 对象传给前端

## 查询多条数据为map集合

### 方法一
```java
/**  
 * 查询所有用户信息为map集合  
 * @return  
 * 将表中的数据以map集合的方式查询，一条数据对应一个map；若有多条数据，就会产生多个map集合，此时可以将这些map放在一个list集合中获取  
 */  
List<Map<String, Object>> getAllUserToMap();
```
```xml
<!--Map<String, Object> getAllUserToMap();-->  
<select id="getAllUserToMap" resultType="map">  
	select * from t_user  
</select>
<!--
	结果：
	[{password=123456, sex=男, id=1, age=23, username=admin},
	{password=123456, sex=男, id=2, age=23, username=张三},
	{password=123456, sex=男, id=3, age=23, username=张三}]
-->
```
### 方法二
```java
/**
 * 查询所有用户信息为map集合
 * @return
 * 将表中的数据以map集合的方式查询，可以一条数据对应一个value；若有多条数据，就会产生多个value的集合，并且最终要以一个map的方式返回数据，此时需要通过@MapKey注解设置map集合的键，值是每条数据
 */
@MapKey("id")
Map<String, Object> getAllUserToMap();
```
```xml
<!--Map<String, Object> getAllUserToMap();-->
<select id="getAllUserToMap" resultType="map">
	select * from t_user
</select>
<!--
	结果：
	{
	1={password=123456, sex=男, id=1, age=23, username=admin},
	2={password=123456, sex=男, id=2, age=23, username=张三},
	3={password=123456, sex=男, id=3, age=23, username=张三}
	}
-->
```
> 注意方法二中的@MapKey ,一定要选取一个唯一的查询出来的属性，因此我们一般会选用主键来作为这个唯一的key

# 特殊SQL的执行

我们上面说过我们可以使用#{} 或者${} 来获取参数 的值，而且我们说能用#{}就不要用${}

下面我们看一下什么情况下直接使用#{} 会有问题

## 模糊查询
```java
/**
 * 根据用户名进行模糊查询
 * @param username 
 * @return java.util.List<com.atguigu.mybatis.pojo.User>
 * @date 2022/2/26 21:56
 */
List<User> getUserByLike(@Param("username") String username);
```
```xml
<!--List<User> getUserByLike(@Param("username") String username);-->
<select id="getUserByLike" resultType="User">
	<!--select * from t_user where username like '%${username}%'-->  
	<!--select * from t_user where username like concat('%',#{username},'%')-->  
	select * from t_user where username like "%"#{username}"%"
</select>
```
- 其中`select * from t_user where username like "%"#{mohu}"%"`是最常用的

> 注意如果我们直接使用like '#{username}' 会产生哪些问题呢？
>
> 因为我们的like 后面必须要在 ' ' 一对单引号中，如果我们在单引号中直接 使用#{ } 这种形式的话，使用的是占位符的形式，但是底层在赋值的时候因为是在单引号中，会当成一个字符串，所以不会为我们的单引号进行赋值，所以就会出现问题 。因此我们可以使用"%"#{mohu}"%" 来拼接一下

## in 关键字
- 只能使用\${}，如果使用#{}，则解析后的sql语句为`delete from t_user where id in ('1,2,3')`，这样是将`1,2,3`看做是一个整体，只有id为`1,2,3`的数据会被删除。正确的语句应该是`delete from t_user where id in (1,2,3)`，或者`delete from t_user where id in ('1','2','3')`

那么对应在mybatis 中，我们只能使用${} 这种形式。我们之前说过如果使用#{} 这样的话，会使用占位符的形式同时会默认给我们加上' ' 单引号，这样就变成了in ('1,2,3') ，这样就变成了在一个字符串值的范围中删除就不对了

```java
/**
 * 根据id批量删除
 * @param ids 
 * @return int
 * @date 2022/2/26 22:06
 */
int deleteMore(@Param("ids") String ids);
```
```xml
<delete id="deleteMore">
	delete from t_user where id in (${ids})
</delete>
```
```java
//测试类
@Test
public void deleteMore() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	SQLMapper mapper = sqlSession.getMapper(SQLMapper.class);
	int result = mapper.deleteMore("1,2,3,8");
	System.out.println(result);
}
```
## 动态设置表名
- 只能使用${}，因为表名不能加单引号，所以不能使用#{}
```java
/**
 * 查询指定表中的数据
 * @param tableName 
 * @return java.util.List<com.atguigu.mybatis.pojo.User>
 * @date 2022/2/27 14:41
 */
List<User> getUserByTable(@Param("tableName") String tableName);
```
```xml
<!--List<User> getUserByTable(@Param("tableName") String tableName);-->
<select id="getUserByTable" resultType="User">
	select * from ${tableName}
</select>
```
## 添加功能获取自增的主键
- 使用场景：一对多或者多对多的添加，当添加完一个表之后要得到我们刚才添加的主键id 值，然后再新添加表的时候使用这个id值，如果这个id值是自增的我们只能再插入完一个表之后获取。

	- t_clazz(clazz_id,clazz_name)   班级表
	- t_student(student_id,student_name,clazz_id) 学生表  
	1. 添加班级信息  （表1）
	2. 获取新添加的班级的id  
	3. 为班级分配学生，即将某学的班级id修改为新添加的班级的id（表2）
	
- 在mapper.xml中设置两个属性

	- useGeneratedKeys：设置使用自增的主键  
	* keyProperty：因为增删改有统一的返回值是受影响的行数，因此只能将获取的自增的主键放在传输的参数user对象的某个属性中
```java
/**
 * 添加用户信息
 * @param user 
 * @date 2022/2/27 15:04
 */
void insertUser(User user);
```
```xml
<!--void insertUser(User user);-->
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
	insert into t_user values (null,#{username},#{password},#{age},#{sex},#{email})
</insert>
```
```java
//测试类
@Test
public void insertUser() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	SQLMapper mapper = sqlSession.getMapper(SQLMapper.class);
	User user = new User(null, "ton", "123", 23, "男", "123@321.com");
	mapper.insertUser(user);
	System.out.println(user);
	//输出：user{id=10, username='ton', password='123', age=23, sex='男', email='123@321.com'}，自增主键存放到了user的id属性中
}
```
