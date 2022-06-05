> 视频教程来自 尚硅谷宋红康老师：https://www.bilibili.com/video/BV1eJ411c7rf?from=search&seid=9990183943173115276&spm_id_from=333.337.0.0

## 第1章：JDBC概述

### 1.1 数据的持久化

- 持久化(persistence)：**把数据保存到可掉电式存储设备中以供之后使用**。大多数情况下，特别是企业级应用，**数据持久化意味着将内存中的数据保存到硬盘**上加以”固化”**，而持久化的实现过程大多通过各种关系数据库来完成**。

- 持久化的主要应用是将内存中的数据存储在关系型数据库中，当然也可以存储在磁盘文件、XML数据文件中。

  ![1566741430592](https://image.imxyu.cn/file/1566741430592.png) 

### 1.2 Java中的数据存储技术

- 在Java中，数据库存取技术可分为如下几类：
  - **JDBC**直接访问数据库
  - JDO (Java Data Object )技术

  - **第三方O/R工具**，如Hibernate, Mybatis 等

- JDBC是java访问数据库的基石，JDO、Hibernate、MyBatis等只是更好的封装了JDBC。

### 1.3 JDBC介绍

- JDBC(Java Database Connectivity)是一个**独立于特定数据库管理系统、通用的SQL数据库存取和操作的公共接口**（一组API），定义了用来访问数据库的标准Java类库，（**java.sql,javax.sql**）使用这些类库可以以一种**标准**的方法、方便地访问数据库资源。
- JDBC为访问不同的数据库提供了一种**统一的途径**，为开发者屏蔽了一些细节问题。
- JDBC的目标是使Java程序员使用JDBC可以连接任何**提供了JDBC驱动程序**的数据库系统，这样就使得程序员无需对特定的数据库系统的特点有过多的了解，从而大大简化和加快了开发过程。
- 如果没有JDBC，那么Java程序访问数据库时是这样的：

![1555575760234](https://image.imxyu.cn/file/1555575760234.png)

***

- 有了JDBC，Java程序访问数据库时是这样的：


![1555575981203](https://image.imxyu.cn/file/1555575981203.png)

***

- 总结如下：

![1566741692804](https://image.imxyu.cn/file/1566741692804.png)

### 1.4 JDBC体系结构

- JDBC接口（API）包括两个层次：
  - **面向应用的API**：Java API，抽象接口，供应用程序开发人员使用（连接数据库，执行SQL语句，获得结果）。
  -  **面向数据库的API**：Java Driver API，供开发商开发数据库驱动程序用。

> **JDBC是sun公司提供一套用于数据库操作的接口，java程序员只需要面向这套接口编程即可。**
>
> **不同的数据库厂商，需要针对这套接口，提供不同实现。不同的实现的集合，即为不同数据库的驱动。																————面向接口编程**

### 1.5 JDBC程序编写步骤

![1565969323908](https://image.imxyu.cn/file/1565969323908.png)

> 补充：ODBC(**Open Database Connectivity**，开放式数据库连接)，是微软在Windows平台下推出的。使用者在程序中只需要调用ODBC API，由 ODBC 驱动程序将调用转换成为对特定的数据库的调用请求。

## 第2章：获取数据库连接

### Driver接口介绍

- java.sql.Driver 接口是所有 JDBC 驱动程序需要实现的接口。这个接口是提供给数据库厂商使用的，不同数据库厂商提供不同的实现。

- 在程序中不需要直接去访问实现了 Driver 接口的类，而是由驱动程序管理器类(java.sql.DriverManager)去调用这些Driver实现。
  - Oracle的驱动：**oracle.jdbc.driver.OracleDriver**
  - mySql的驱动： **com.mysql.jdbc.Driver**



- 在项目文件下创建Lib文件夹

- 拷贝驱动文件到LIb文件夹下

  ![Untitled](https://image.imxyu.cn/file/Untitled.png)

  

![Untitled (1)](https://image.imxyu.cn/file/Untitled (1).png)

### 2.4 数据库连接方式举例

#### 2.4.1 连接方式一

```java
	@Test
    public void testConnection1() {
        try {
            //1.提供java.sql.Driver接口实现类的对象
            Driver driver = null;
            driver = new com.mysql.jdbc.Driver();

            //2.提供url，指明具体操作的数据
            String url = "jdbc:mysql://localhost:3306/test";

            //3.提供Properties的对象，指明用户名和密码
            Properties info = new Properties();
            info.setProperty("user", "root");
            info.setProperty("password", "abc123");

            //4.调用driver的connect()，获取连接
            Connection conn = driver.connect(url, info);
            System.out.println(conn);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```

> 缺点说明：上述代码中显式出现了第三方数据库的API，new com.mysql.jdbc.Driver();这里

#### 2.4.2 连接方式二

```java
	@Test
    public void testConnection2() {
        try {
            //1.实例化Driver
            String className = "com.mysql.jdbc.Driver";
            Class clazz = Class.forName(className);
            Driver driver = (Driver) clazz.newInstance();

            //2.提供url，指明具体操作的数据
            String url = "jdbc:mysql://localhost:3306/test";

            //3.提供Properties的对象，指明用户名和密码
            Properties info = new Properties();
            info.setProperty("user", "root");
            info.setProperty("password", "abc123");

            //4.调用driver的connect()，获取连接
            Connection conn = driver.connect(url, info);
            System.out.println(conn);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

> 说明：相较于方式一，这里使用反射实例化Driver，不在代码中体现第三方数据库的API。体现了面向接口编程思想。
>
> 因为第一种方式直接new 的第三方jar包的类，不具备可移植性，如果想换oracle 需要改动这里的代码。而要想有可移植性就不能出现第三方的jar包，所以这里使用java的方法反射创建对象，反射中传入的是字符串，这样就可以更好的扩展。

#### 2.4.3 连接方式三

```java
	@Test
    public void testConnection3() {
        try {
            //1.数据库连接的4个基本要素：
            String url = "jdbc:mysql://localhost:3306/test";
            String user = "root";
            String password = "abc123";
            String driverName = "com.mysql.jdbc.Driver";

            //2.实例化Driver
            Class clazz = Class.forName(driverName);
            Driver driver = (Driver) clazz.newInstance();
            //3.注册驱动
            DriverManager.registerDriver(driver);
            //4.获取连接
            Connection conn = DriverManager.getConnection(url, user, password);
            System.out.println(conn);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
```

> 说明：使用DriverManager实现数据库的连接。体会获取连接必要的4个基本要素。（driver驱动对象、url、user 、pass）

#### 2.4.4 连接方式四

```java
	@Test
    public void testConnection4() {
        try {
            //1.数据库连接的4个基本要素：
            String url = "jdbc:mysql://localhost:3306/test";
            String user = "root";
            String password = "abc123";
            String driverName = "com.mysql.jdbc.Driver";

            //2.加载驱动 （①实例化Driver ②注册驱动）

            Class.forName(driverName);


            //Driver driver = (Driver) clazz.newInstance();
            //3.注册驱动
            //DriverManager.registerDriver(driver);
            /*
            可以注释掉上述代码的原因，是因为在mysql的驱动实现类中声明有这个静态代码块，当类加载的时候会调用这句：
          
            
            public class Driver extends NonRegisteringDriver implements java.sql.Driver {
  
            static {
                try {
                    java.sql.DriverManager.registerDriver(new Driver());
                } catch (SQLException E) {
                    throw new RuntimeException("Can't register driver!");
                }
            }

             */


            //3.获取连接
            Connection conn = DriverManager.getConnection(url, user, password);
            System.out.println(conn);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
```

> 说明：不必显式的注册驱动了。因为在DriverManager的源码中已经存在静态代码块，实现了驱动的注册。
>
>       //但是发现省略这一步也还可以，原因是在引入jar包的时候有一个文件，在这里面会写了路径。但是我们还是不要省略这句
>       因为只有mysql帮我们写了，如果oracle，还是没有。需要我们去加载。所以不要省略这句
>             Class.forName(driverName);

![image-20220307195640184](https://image.imxyu.cn/file/image-20220307195640184.png)

#### 2.4.5 连接方式五(使用配置文件读取，开发常用）

使用配置文件数据库连接信息

```java
	@Test
    public  void testConnection5() throws Exception {
    	//1.加载配置文件
        InputStream is = ConnectionTest.class.getClassLoader().getResourceAsStream("jdbc.properties");
        Properties pros = new Properties();
        pros.load(is);
        
        //2.读取配置信息
        String user = pros.getProperty("user");
        String password = pros.getProperty("password");
        String url = pros.getProperty("url");
        String driverClass = pros.getProperty("driverClass");

        //3.加载驱动
        Class.forName(driverClass);

        //4.获取连接
        Connection conn = DriverManager.getConnection(url,user,password);
        System.out.println(conn);

    }
```

> ConnectionTest.class.getClassLoader().getResourceAsStream("jdbc.properties");
>
> 这里是根据我们的类获取类加载器，因此是系统的类加载器，我们也可以用一种简单的方法：
>
> 直接ClassLoader.getSystemClassLoader().getResourceAsStream   。这样也可以直接获得系统的类加载器

其中，配置文件声明在工程的src目录下：【jdbc.properties】 ，放在项目中可以打包的时候包含我们这个文件

```properties
user=root
password=abc123
url=jdbc:mysql://localhost:3306/test
driverClass=com.mysql.jdbc.Driver
```

> 说明：使用配置文件的方式保存配置信息，在代码中加载配置文件
>
> **使用配置文件的好处：**
>
> ①实现了代码和数据的分离，实现了解耦，如果需要修改配置信息，直接在配置文件中修改，不需要深入代码
> ②如果修改了配置信息，省去重新编译的过程。

## 第3章：使用PreparedStatement实现CRUD操作

### 3.1 操作和访问数据库

- 数据库连接被用于向数据库服务器发送命令和 SQL 语句，并接受数据库服务器返回的结果。其实一个数据库连接就是一个Socket连接。

- 在 java.sql 包中有 3 个接口分别定义了对数据库的调用的不同方式：
  - Statement：用于执行静态 SQL 语句并返回它所生成结果的对象。 
  - PrepatedStatement：SQL 语句被预编译并存储在此对象中，可以使用此对象多次高效地执行该语句。
  - CallableStatement：用于执行 SQL 存储过程

  > PrepatedStatement和CallableStatement都是Statement的**子接口**
  
  ![1566573842140](https://image.imxyu.cn/file/1566573842140.png)

### 3.2 使用Statement操作数据表的弊端

- 通过调用 Connection 对象的 createStatement() 方法创建该对象。该对象用于执行静态的 SQL 语句，并且返回执行结果。

- Statement 接口中定义了下列方法用于执行 SQL 语句：

  ```sql
  int excuteUpdate(String sql)：执行更新操作INSERT、UPDATE、DELETE
  ResultSet executeQuery(String sql)：执行查询操作SELECT
  ```

- 但是使用Statement操作数据表存在弊端：

  - **问题一：存在拼串操作，繁琐**

  ```java
  String sql = "SELECT user,password FROM user_table WHERE USER = '" + userName + "' AND PASSWORD = '" + password + "'";
  ```

  - **问题二：存在SQL注入问题**

- SQL 注入是利用某些系统没有对用户输入的数据进行充分的检查，而在用户输入数据中注入非法的 SQL 语句段或命令(如：SELECT user, password FROM user_table WHERE user='a' OR 1 = ' AND password = ' OR '1' = '1') ，从而利用系统的 SQL 引擎完成恶意行为的做法。

  > 这样就变成了3个or 连接，1 = ' AND password = '  这个就变成了是一个字符串和1的判断，最后一个or  1=1恒成立的

- 对于 Java 而言，要防范 SQL 注入，只要用 PreparedStatement(从Statement扩展而来) 取代 Statement 就可以了。

- statement代码演示(不需要掌握)：

```java
public class StatementTest {

	// 使用Statement的弊端：需要拼写sql语句，并且存在SQL注入的问题
	@Test
	public void testLogin() {
		Scanner scan = new Scanner(System.in);

		System.out.print("用户名：");
		String userName = scan.nextLine();
		System.out.print("密   码：");
		String password = scan.nextLine();

		// SELECT user,password FROM user_table WHERE USER = '1' or ' AND PASSWORD = '='1' or '1' = '1';
		String sql = "SELECT user,password FROM user_table WHERE USER = '" + userName + "' AND PASSWORD = '" + password
				+ "'";
		User user = get(sql, User.class);
		if (user != null) {
			System.out.println("登陆成功!");
		} else {
			System.out.println("用户名或密码错误！");
		}
	}

	// 使用Statement实现对数据表的查询操作
	public <T> T get(String sql, Class<T> clazz) {
		T t = null;

		Connection conn = null;
		Statement st = null;
		ResultSet rs = null;
		try {
			// 1.加载配置文件
			InputStream is = StatementTest.class.getClassLoader().getResourceAsStream("jdbc.properties");
			Properties pros = new Properties();
			pros.load(is);

			// 2.读取配置信息
			String user = pros.getProperty("user");
			String password = pros.getProperty("password");
			String url = pros.getProperty("url");
			String driverClass = pros.getProperty("driverClass");

			// 3.加载驱动
			Class.forName(driverClass);

			// 4.获取连接
			conn = DriverManager.getConnection(url, user, password);

			st = conn.createStatement();

			rs = st.executeQuery(sql);

			// 获取结果集的元数据
			ResultSetMetaData rsmd = rs.getMetaData();

			// 获取结果集的列数
			int columnCount = rsmd.getColumnCount();

			if (rs.next()) {

				t = clazz.newInstance();

				for (int i = 0; i < columnCount; i++) {
					// //1. 获取列的名称
					// String columnName = rsmd.getColumnName(i+1);

					// 1. 获取列的别名
					String columnName = rsmd.getColumnLabel(i + 1);

					// 2. 根据列名获取对应数据表中的数据
					Object columnVal = rs.getObject(columnName);

					// 3. 将数据表中得到的数据，封装进对象
					Field field = clazz.getDeclaredField(columnName);
					field.setAccessible(true);
					field.set(t, columnVal);
				}
				return t;
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			// 关闭资源
			if (rs != null) {
				try {
					rs.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
			if (st != null) {
				try {
					st.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}

			if (conn != null) {
				try {
					conn.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
		}

		return null;
	}
}
```

综上：

![1566569819744](https://image.imxyu.cn/file/1566569819744.png)

### 3.3 PreparedStatement的使用

#### 3.3.1 PreparedStatement介绍

- 可以通过调用 Connection 对象的 **preparedStatement(String sql)** 方法获取 PreparedStatement 对象

- **PreparedStatement 接口是 Statement 的子接口，它表示一条预编译过的 SQL 语句**

- PreparedStatement 对象所代表的 SQL 语句中的参数用问号(?)来表示，调用 PreparedStatement 对象的 setXxx() 方法来设置这些参数. setXxx() 方法有两个参数，第一个参数是要设置的 SQL 语句中的参数的索引(从 1 开始)，第二个是设置的 SQL 语句中的参数的值

#### 3.3.2 PreparedStatement vs Statement

- 代码的可读性和可维护性。

- **PreparedStatement 能最大可能提高性能：**
  - DBServer会对**预编译**语句提供性能优化。因为预编译语句有可能被重复调用，所以<u>语句在被DBServer的编译器编译后的执行代码被缓存下来，那么下次调用时只要是相同的预编译语句就不需要编译，只要将参数直接传入编译过的语句执行代码中就会得到执行。</u>
  - 在statement语句中,即使是相同操作但因为数据内容不一样,所以整个语句本身不能匹配,没有缓存语句的意义.事实是没有数据库会对普通语句编译后的执行代码缓存。这样<u>每执行一次都要对传入的语句编译一次。</u>
  - (语法检查，语义检查，翻译成二进制命令，缓存)

- PreparedStatement 可以防止 SQL 注入 

#### 3.3.3 Java与SQL对应数据类型转换表

| Java类型           | SQL类型                  |
| ------------------ | ------------------------ |
| boolean            | BIT                      |
| byte               | TINYINT                  |
| short              | SMALLINT                 |
| int                | INTEGER                  |
| long               | BIGINT                   |
| String             | CHAR,VARCHAR,LONGVARCHAR |
| byte   array       | BINARY  ,    VAR BINARY  |
| java.sql.Date      | DATE                     |
| java.sql.Time      | TIME                     |
| java.sql.Timestamp | TIMESTAMP                |

#### 3.3.4 使用PreparedStatement实现增、删、改操作

##### 增加操作

```java
//    向customers表中插入一条数据
    public void insertTest()  {
//        1.读取配置信息
        Connection conn=null;
        PreparedStatement ps=null;
        try{
            ResourceBundle rb=ResourceBundle.getBundle("jdbc");
            String url=rb.getString("url");
            String user= rb.getString("user");
            String password=rb.getString("password");
            String driverClass=rb.getString("driverClass");
//        2.加载驱动
            Class.forName(driverClass);
//        3.建立连接
            conn=  DriverManager.getConnection(url,user,password);
        System.out.println(conn);
//        conn.commit();
//        4.预编译sql语句，返回preparedStatement实例
            String sql="insert into customers(name,email,birth)values(?,?,?)";
            ps=conn.prepareStatement(sql);
//        5.填充占位符,下标从1开始
            ps.setString(1,"sad");
            ps.setString(2,"sad@qq.com");
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd");
            String st="2021-10-25";
            //首先使用java的Date 类，转换格式
            java.util.Date d=sdf.parse(st);
            //然后通过java的Date类的getTime 获取时间然后转换成java.sql 包下的Date 
            ps.setDate(3,new java.sql.Date(d.getTime()));
       // 6.执行操作
            ps.execute();
        }catch (Exception e){ //这里要用try catch ,因为最后finally 要关闭连接，不能直接throw出去
            e.printStackTrace();
        }finally {
       // 7.关闭连接
            //这里注意要进行非空判断，万一上面获取链接出现异常这里获得的为null，所以要判断
            if(ps!=null){
                try {
                    ps.close();
                } catch (SQLException throwables) {
                    throwables.printStackTrace();
                }
            }
            //同上
            if(conn!=null){
                try {
                    conn.close();
                } catch (SQLException throwables) {
                    throwables.printStackTrace();
                }
            }
        }
    }

```

> 为什么是prepareStatement 能解决sql注入问题，因为在PrepareStatement  ps=conn.prepareStatement(sql); 的时候就传入了sql 语句作为参数，这里会有一个预编译，然后再调用 ps.execute()执行。而我们之前使用的	Statement 则没有传入sql，而是执行的时候直接调用sql语句，所以会有问题
>
> 		Statement st = conn.createStatement();
> 			rs = st.executeQuery(sql);



##### 数据库连接和关闭封装成方法

- 每次都要进行数据库的连接与关闭
- 可以将开启关闭操作封装到工具类JDBCUtils
- 同时获取连接获取通过Connection con = DriverManager.getConnection(url, user, password); 封装到一个方法当中即可

```java
public class JDBCUtils {
    //获取连接封装方法
	public static Connection getConnection() throws Exception{
		//通过类加载器 获取 数据库文件中的配置信息
		ClassLoader loader = ClassLoader.getSystemClassLoader();
		InputStream is = loader.getResourceAsStream("jdbc.properties");
		Properties info = new Properties();
		info.load(is);
		String user = info.getProperty("user");
		String password = info.getProperty("password");
		String url = info.getProperty("url");
		String driver = info.getProperty("driver");
		//通过反射加载驱动类
		Class.forName(driver);
		//连接数据库
		Connection con = DriverManager.getConnection(url, user, password);
		return con;
	}

    //关闭资源封装方法
	public static void closeResource(Connection con, Statement ps) {
		try {
			if (ps != null)
				ps.close();
		} catch (SQLException throwables) {
			throwables.printStackTrace();
		}
		try {
			if (con != null)
				con.close();
		} catch (SQLException throwables) {
			throwables.printStackTrace();
		}
	}
}
```

##### 修改操作

下面我们就可以直接调用封装的方法，代码就简化了很多

```java
@Test
	public void updateTest(){
		Connection conn = null;
		PreparedStatement ps = null;
		try {
			//开启连接
			conn = JDBCUtils.getConnection();

			//1.SQL预编译、获取PrepareStatement
			String sql = "update customers set name = ? where id = ?;";
			ps = conn.prepareStatement(sql);

			//2.填充占位符
			ps.setObject(1,"莫扎特11");
			ps.setObject(2,18);

			//3. 执行sql
			ps.execute();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			//关闭连接
			JDBCUtils.closeResource(conn,ps);
		}
	}
```

##### 删除操作

```java
@Test
	public void deleteTest(){
		Connection conn = null;
		PreparedStatement ps = null;
		try {
			//开启连接
			conn = JDBCUtils.getConnection();

			//1.SQL预编译、获取PrepareStatement
			String sql = "delete from customers where id = ?";
			ps = conn.prepareStatement(sql);

			//2.填充占位符
			ps.setObject(1,18);

			//3. 执行sql
			ps.execute();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			//关闭连接
			JDBCUtils.closeResource(conn,ps);
		}
	}
```

##### 增删改通用操作

通过上方增删改的例子，可以看出，**变化的只有SQL和填充的占位符**

所以可以将增删改操作写成一个通用的方法，每次只要传入SQL和可变参数即可

```java
	//通用的增、删、改操作（体现一：增、删、改 ； 体现二：针对于不同的表）
	public void update(String sql,Object ... args){
		Connection conn = null;
		PreparedStatement ps = null;
		try {
			//1.获取数据库的连接
			conn = JDBCUtils.getConnection();
			
			//2.获取PreparedStatement的实例 (或：预编译sql语句)
			ps = conn.prepareStatement(sql);
			//3.填充占位符 ，注意这里是i+1, java都是从1开始
			for(int i = 0;i < args.length;i++){
				ps.setObject(i + 1, args[i]);
			}
			
			//4.执行sql语句
			ps.execute();
		} catch (Exception e) {
			
			e.printStackTrace();
		}finally{
			//5.关闭资源
			JDBCUtils.closeResource(conn, ps);
			
		}
	}
```



#### 3.3.5 使用PreparedStatement实现查询操作

##### 普通查询操作

```java
@Test
	public void queryTest() throws Exception {
		//连接数据库
		Connection conn = JDBCUtils.getConnection();
		//预编译SQL
		String sql = "SELECT id,`NAME`,email,birth FROM customers where id = ?;";
		PreparedStatement ps = conn.prepareStatement(sql);
		//填充占位符
		ps.setObject(1,1);
		//执行，获取结果集
		ResultSet res = ps.executeQuery();

		if(res.next()){ //判断结果集下一条是否有数据，有数据返回Ture，并指针下移，如果返回false，指针不会下移
			int id = res.getInt(1);
			String name = res.getString(2);
			String email = res.getString(3);
			Date birth = res.getDate(4);
			//打印方式一
			System.out.println("id:" + id + " name:" + name + " email:" + email+ " birth:" + birth);

			//打印方式二
			Object[] data = new Object[]{id,name,email,birth};
			System.out.println(Arrays.toString(data));

			//打印方式三
			Customer customer = new Customer(id, name, email, birth);
			System.out.println(customer);
		}
		
		//关闭资源
		JDBCUtils.closeResource(conn,ps,res);
	}
```

- 查询与增删改的区别，查询返回结果集ResultSet

- 使用next方法检查结果集中是否存在数据

- 有数据则指针下移看，指向数据，无数据则指针不下移

  ResultSet.next方法类似iterate.hasNext 与 iterate.next的功能综合版本，但是不会返回数据

- 是通过索引获取当前行的数据

- 推荐使用方式三的方式存储数据：存储到对象之中

##### 针对单表查询操作

下面这个是可以针对特定表的查询，根据返回值利用反射动态封装

```java
	//Customer 表进行查询
public Customer CommonqueryTest(String sql,Object... args) throws Exception {
		//连接数据库
		Connection conn = JDBCUtils.getConnection();
		//预编译SQL
		PreparedStatement ps = conn.prepareStatement(sql);
		//填充占位符
		for (int i = 0; i < args.length; i++) {
			ps.setObject(i+1,args[i]);
		}
		//返回结果集
		ResultSet res = ps.executeQuery();
		//获取结果集的元数据,也就是描述结果的信息比如列的相关信息
		ResultSetMetaData rsmd = res.getMetaData();
		int columnCount = rsmd.getColumnCount();
		//判断结果集中是否存在对象，将指针指向数据行
		if(res.next())
		{
			Customer customer = new Customer();
			for (int i = 0; i < columnCount; i++) {
				//获取列名、值
				Object columnValue = res.getObject(i + 1);
				//String columnName = rsmd.getColumnName(i + 1); //使用getColumnLabel代替
				String columnLabel = rsmd.getColumnLabel(i + 1);
                //反射类字段值
				Field field = Customer.class.getDeclaredField(columnName);
				//访问私有字段
				field.setAccessible(true);
				//设置值
				field.set(customer,columnValue);
			}
			return customer;
		}
		return null;
	}
```

##### 针对通用表操作（最终）

通过反射可以指定任意表，此时我们就需要使用Class 对象，并且Class 对象是支持泛型的，我们可以传入Class<T> 来表示一个不确定的对象类型

```java
// 注意泛型方法声明在返回值前面，表示泛型方法
public <T> List<T> CommonqueryTest3(Class<T> clazz, String sql, Object... args) throws Exception {
		//连接数据库
		Connection conn = JDBCUtils.getConnection();
		//预编译
		PreparedStatement ps = conn.prepareStatement(sql);
		//填充占位符
		for (int i = 0; i < args.length; i++) {
			ps.setObject(i+1,args[i]);
		}
		//查询获取结果集
		ResultSet res = ps.executeQuery();
		//获取结果集的元数据,也就是描述结果的信息比如列的相关信息
		ResultSetMetaData rsmd = res.getMetaData();
		int columnCount = rsmd.getColumnCount();
		ArrayList<T> list = new ArrayList<>();
		//获取结果集，这里我们直接获取多个行，所以使用while循环，list封装
		while(res.next())
		{
			T t = clazz.newInstance();
			for (int i = 0; i < columnCount; i++) {
				//获取列名、值
				Object columnValue = res.getObject(i + 1);
			//	String columnName = rsmd.getColumnName(i + 1);
                String columnLabel = rsmd.getColumnLabel(i + 1);
				//反射类字段值
				Field field = Customer.class.getDeclaredField(columnName);
				//访问私有字段
				field.setAccessible(true);
				//设置值
				field.set(t,columnValue);
			}
			list.add(t);
		}
		return list;
	}
```

> 这里还需要提醒一点，就是在java 中的索引都是从1开始的，所以对于上面这些java 函数，包括列名什么的索引都是从1开始

![image-20220308201129801](https://image.imxyu.cn/file/image-20220308201129801.png)

>  这里因为数据库中的表列名都是下划线的，而我们的java 对象中通常喜欢写成 orderId, orderName ,orderData , 这种大写的
>
> 所以对应的sql我们查询出来的时候我们就要为他们起上别名.比如
>
> select order_id orderId, order_name orderName ,order_data orderData from order 
>
> 这样我们查询出来的列才能正确的通过orm 映射到我们的pojo 的相应字段中去，这里注意，如果我们使用了别名，我们通过元数据ResultSetMetaData获取列的信息的时候只能使用getColumnLabel方法，这个方法才是获取别名的信息，而不能使用getColumnName ，这样获取的还是原始的列名
>
> 结论： 以后都使用getColumnLabel（）方法获取列名， 默认的如果我们没有用列名获取的还是原始列的名，所以不会有任何问题。更加通用

##### 总结

数据库字段名要与Java类中属性名一致

- 根据ORM编程思想，数据库一行数据对应一个Java对象
- 数据库一个字段对应Java类的一个属性
- 所以当数据库字段与Java属性名不一致会报错

数据库字段名要与Java类中属性名不一致：则必须在SQL语句中给字段起别名

 ⭐ 结果集元数据获取列名时不要使用getColumnName方法，而是使用getColumnLabel代替 getColumnName：获取字段名 getColumnLabel：先获取别名，没有别名获取字段名



### 3.4 ResultSet与ResultSetMetaData

#### 3.4.1 ResultSet

- 查询需要调用PreparedStatement 的 executeQuery() 方法，查询结果是一个ResultSet 对象

- ResultSet 对象以逻辑表格的形式封装了执行数据库操作的结果集，ResultSet 接口由数据库厂商提供实现
- ResultSet 返回的实际上就是一张数据表。有一个指针指向数据表的第一条记录的前面。

- ResultSet 对象维护了一个指向当前数据行的**游标**，初始的时候，游标在第一行之前，可以通过 ResultSet 对象的 next() 方法移动到下一行。调用 next()方法检测下一行是否有效。若有效，该方法返回 true，且指针下移。相当于Iterator对象的 hasNext() 和 next() 方法的结合体。
- 当指针指向一行时, 可以通过调用 getXxx(int index) 或 getXxx(int columnName) 获取每一列的值。

  - 例如: getInt(1), getString("name")
  - **注意：Java与数据库交互涉及到的相关Java API中的索引都从1开始。**

- ResultSet 接口的常用方法：
  - boolean next()

  - getString()
  - …

  ![1555580152530](https://image.imxyu.cn/file/1555580152530.png)

#### 3.4.2 ResultSetMetaData

- 可用于获取关于 ResultSet 对象中列的类型和属性信息的对象

- ResultSetMetaData meta = rs.getMetaData();
  - **getColumnName**(int column)：获取指定列的名称
  - **getColumnLabel**(int column)：获取指定列的别名
  - **getColumnCount**()：返回当前 ResultSet 对象中的列数。 

  - getColumnTypeName(int column)：检索指定列的数据库特定的类型名称。 
  - getColumnDisplaySize(int column)：指示指定列的最大标准宽度，以字符为单位。 
  - **isNullable**(int column)：指示指定列中的值是否可以为 null。 

  -  isAutoIncrement(int column)：指示是否自动为指定列进行编号，这样这些列仍然是只读的。 

![1555579494691](https://image.imxyu.cn/file/1555579494691.png)

**问题1：得到结果集后, 如何知道该结果集中有哪些列 ？ 列名是什么？**

​     需要使用一个描述 ResultSet 的对象， 即 ResultSetMetaData

**问题2：关于ResultSetMetaData**

1. **如何获取 ResultSetMetaData**： 调用 ResultSet 的 getMetaData() 方法即可
2. **获取 ResultSet 中有多少列**：调用 ResultSetMetaData 的 getColumnCount() 方法
3. **获取 ResultSet 每一列的列的别名是什么**：调用 ResultSetMetaData 的getColumnLabel() 方法

![1555579816884](https://image.imxyu.cn/file/1555579816884.png)

### 3.5 资源的释放

- 释放ResultSet, Statement,Connection。
- 数据库连接（Connection）是非常稀有的资源，用完后必须马上释放，如果Connection不能及时正确的关闭将导致系统宕机。Connection的使用原则是**尽量晚创建，尽量早的释放。**
- 可以在finally中关闭，保证及时其他代码出现异常，资源也一定能被关闭。





### 3.6 JDBC API小结

- 两种思想
  - 面向接口编程的思想

  - ORM思想(object relational mapping)
    - 一个数据表对应一个java类
    - 表中的一条记录对应java类的一个对象
    - 表中的一个字段对应java类的一个属性

  > sql是需要结合列名和表的属性名来写。注意起别名。

- 两种技术
  - JDBC结果集的元数据：ResultSetMetaData
    - 获取列数：getColumnCount()
    - 获取列的别名：getColumnLabel()
  - 通过反射，创建指定类的对象，获取指定的属性并赋值



## PreparedStatement对比Statement

- 拼串和SQL注入的问题

```sql
# 1.Statement中，直接输入SQL语句进行执行，就会出现SQL注入问题
SELECT USER,PASSWORD FROM user_table WHERE USER = '1' OR ' AND PASSWORD = '='1' OR '1' = '1'

# 2.在PreparStatement中则是先进行SQL的预编译, 使用占位符，不用拼串，后续填充占位符，也不会改变SQL原有的逻辑，解决SQL注入
String sql = "SELECT * FROM admin WHERE username = ? AND PASSWORD = ?;";
PreparedStatement ps = conn.prepareStatement(sql);
```

- **Statement无法操作Blob类型的数据**
- prepaerStatement便于批量操作，

Statement批量操作：执行一次SQL语句，需要校验一次SQL

prepaerStatement批量操作：无论执行多少次SQL，都只校验一次，效率更高

- **PreparedStatement 能最大可能提高性能：**

DBServer会对**预编译**语句提供性能优化。因为预编译语句有可能被重复调用，所以语句在被DBServer的编译器编译后的执行代码被缓存下来，那么下次调用时只要是相同的预编译语句就不需要编译，只要将参数直接传入编译过的语句执行代码中就会得到执行。

在statement语句中,即使是相同操作但因为数据内容不一样,所以整个语句本身不能匹配,没有缓存语句的意义.事实是没有数据库会对普通语句编译后的执行代码缓存。这样每执行一次都要对传入的语句编译一次。

(语法检查，语义检查，翻译成二进制命令，缓存)



## 第4章 PreparedStatement操作BLOB类型字段

### 4.1 MySQL BLOB类型

- MySQL中，BLOB是一个二进制大型对象，是一个可以存储大量数据的容器，它能容纳不同大小的数据。
- 插入BLOB类型的数据必须使用PreparedStatement，因为BLOB类型的数据无法使用字符串拼接写的。

- MySQL的四种BLOB类型(除了在存储的最大信息量上不同外，他们是等同的)

![1555581069798](https://image.imxyu.cn/file/1555581069798.png)

- 实际使用中根据需要存入的数据大小定义不同的BLOB类型。
- 需要注意的是：如果存储的文件过大，数据库的性能会下降。
- 如果在指定了相关的Blob类型以后，还报错：xxx too large，那么在mysql的安装目录下，找my.ini文件加上如下的配置参数： **max_allowed_packet=16M**。同时注意：修改了my.ini文件之后，需要重新启动mysql服务。

### 4.2 向数据表中插入大数据类型

```java
//获取连接
Connection conn = JDBCUtils.getConnection();
		
String sql = "insert into customers(name,email,birth,photo)values(?,?,?,?)";
PreparedStatement ps = conn.prepareStatement(sql);

// 填充占位符
ps.setString(1, "徐海强");
ps.setString(2, "xhq@126.com");
ps.setDate(3, new Date(new java.util.Date().getTime()));
// 操作Blob类型的变量
FileInputStream fis = new FileInputStream("xhq.png");
ps.setBlob(4, fis);
//执行
ps.execute();
		
fis.close();
JDBCUtils.closeResource(conn, ps);

```



### 4.3 修改数据表中的Blob类型字段

```java
Connection conn = JDBCUtils.getConnection();
String sql = "update customers set photo = ? where id = ?";
PreparedStatement ps = conn.prepareStatement(sql);

// 填充占位符
// 操作Blob类型的变量
FileInputStream fis = new FileInputStream("coffee.png");
ps.setBlob(1, fis);
ps.setInt(2, 25);

ps.execute();

fis.close();
JDBCUtils.closeResource(conn, ps);
```



### 4.4 从数据表中读取大数据类型

```java
String sql = "SELECT id, name, email, birth, photo FROM customer WHERE id = ?";
conn = getConnection();
ps = conn.prepareStatement(sql);
ps.setInt(1, 8);
rs = ps.executeQuery();
if(rs.next()){
	Integer id = rs.getInt(1);
    String name = rs.getString(2);
	String email = rs.getString(3);
    Date birth = rs.getDate(4);
	Customer cust = new Customer(id, name, email, birth);
    System.out.println(cust); 
    //读取Blob类型的字段
	Blob photo = rs.getBlob(5);
    //这里读取完之后存入到本地文件
	InputStream is = photo.getBinaryStream();
	OutputStream os = new FileOutputStream("c.jpg");
	byte [] buffer = new byte[1024];
	int len = 0;
	while((len = is.read(buffer)) != -1){
		os.write(buffer, 0, len);
	}
    JDBCUtils.closeResource(conn, ps, rs);
		
	if(is != null){
		is.close();
	}
		
	if(os !=  null){
		os.close();
	}
    
}

```



## 第5章 批量插入

接下来我们看一下prepaeStatement 相对于statement的另一个优势，就是支持batch 批量操作

### 5.1 批量执行SQL语句

当需要成批插入或者更新记录时，可以采用Java的批量**更新**机制，这一机制允许多条语句一次性提交给数据库批量处理。通常情况下比单独提交处理更有效率

JDBC的批量处理语句包括下面三个方法：
- **addBatch(String)：添加需要批量处理的SQL语句或是参数；**
- **executeBatch()：执行批量处理语句；**
- **clearBatch():清空缓存的数据**

通常我们会遇到两种批量执行SQL语句的情况：
- 多条SQL语句的批量处理；
- 一个SQL语句的批量传参；



### 5.2 高效的批量插入

举例：向数据表中插入20000条数据

- 数据库中提供一个goods表。创建如下：

```sql
CREATE TABLE goods(
id INT PRIMARY KEY AUTO_INCREMENT,
NAME VARCHAR(20)
);
```



#### 方式一: 使用Statement进行批量添加



```sql
@Test
	public void test3() throws Exception{
		Connection conn = JDBCUtils.getConnection();
		Statement st = conn.createStatement();
		long start = System.currentTimeMillis();
		for (int i = 0; i < 100000; i++) {
			String sql = "INSERT INTO goods(NAME) VALUES('Jack_"+i+"')";
			st.execute(sql);
		}
		long end = System.currentTimeMillis();
		System.out.println(end-start);
		JDBCUtils.closeResource(conn,st);
	}

------------------
142333
```

#### 方式二:使用PrepaerStatement进行批量添加

```sql
@Test
	public void test4() throws Exception{
		Connection conn = JDBCUtils.getConnection();
		String sql = "INSERT INTO goods(NAME) VALUES(?);";
		PreparedStatement ps = conn.prepareStatement(sql);
		long start = System.currentTimeMillis();
		for (int i = 0; i < 100000; i++) {
			ps.setObject(1,"jack_"+i);
			ps.execute();
		}
		long end = System.currentTimeMillis();
		System.out.println(end-start);
		JDBCUtils.closeResource(conn,ps);
	}
--------------------
142134
```

> 💡 对比方式一和方式二 
>
> 方式一：并且从jvm的角度来看，方式一在每次for循环的时候都要创建一个字符串对象，并且每次都会创建新的sql语句，对SQL进行检查 
>
> 方式二：而对于方式二只需要在外层创建一个sql对象，并且PrepaerStatement会创建SQL缓存，不会多次创建，下一次会缓存原有的sql语句只会在参数上进行不同的设置即可，只对SQL检查一次 
>
> 总结：理论上方式二在大批量数据操作上，效率更高，但是运行时间差不多

#### 方式三:PrepaerStatement的batch批量操作

方式一和方式二，每一条SQL都会去执行一次，效率比较慢

方式三通过将需要执行的SQL缓存下来，批量执行，提升效率

- 使用 addBatch() / executeBatch() / clearBatch()
- mysql服务器默认是关闭批处理的，我们需要通过一个参数，让mysql开启批处理的支持。`?rewriteBatchedStatements=true 写在配置文件的url后面`

```java
@Test
	public void test5() throws Exception{
		Connection conn = JDBCUtils.getConnection();
		String sql = "INSERT INTO goods(NAME) VALUES(?)";
		PreparedStatement ps = conn.prepareStatement(sql);
		long start = System.currentTimeMillis();
		for (int i = 1; i <= 100000; i++) {
			ps.setObject(1,"jack_"+i);
			//缓存SQL
			ps.addBatch();
			if(i%500 == 0){
				//将缓存的SQL批量执行
				ps.executeBatch();
				//清理缓存
				ps.clearBatch();
			}
		}
		long end = System.currentTimeMillis();
		System.out.println(end-start);
		JDBCUtils.closeResource(conn,ps);
	}
--------------------
1374
```

>  💡 MySQL5.1.37之前版本不支持批量处理
>
> 方式二插入10W数据：142134毫秒
>
> 方式三插入10W条数据：1377毫秒
>
> 方式三插入100W条数据：11602毫秒



#### 方式四: 手动提交事务

在mysql中对于DML 语句默认是自动提交事务的，也就是说每一句增删改都会直接执行。

在方式三的基础上再次进行迭代 将自动提交事务关闭，全部SQL执行完毕之后再提交

```java
@Test
	public void test6() throws Exception{
		Connection conn = JDBCUtils.getConnection();
		//关闭事务自动提交
		conn.setAutoCommit(false);
		String sql = "INSERT INTO goods(NAME) VALUES(?)";
		PreparedStatement ps = conn.prepareStatement(sql);
		long start = System.currentTimeMillis();
		for (int i = 1; i <= 1000000; i++) {
			ps.setObject(1,"jack_"+i);
			ps.addBatch();
			if(i%500 == 0){
				ps.executeBatch();
				ps.clearBatch();
			}
		}
		//提交事务
		conn.commit();
		long end = System.currentTimeMillis();
		System.out.println(end-start);
		JDBCUtils.closeResource(conn,ps);
	}

---------------------
//插入100w数据
8730
```

 💡 方式四效率最高
