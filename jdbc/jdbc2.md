## 第6章： 数据库事务

### 6.1 数据库事务介绍

- **事务：一组逻辑操作单元,使数据从一种状态变换到另一种状态。**
- **事务处理（事务操作）：**保证所有事务都作为一个工作单元来执行，即使出现了故障，都不能改变这种执行方式。当在一个事务中执行多个操作时，要么所有的事务都**被提交(commit)**，那么这些修改就永久地保存下来；要么数据库管理系统将放弃所作的所有修改，整个事务**回滚(rollback)**到最初状态。
- 为确保数据库中数据的**一致性**，数据的操纵应当是离散的成组的逻辑单元：当它全部完成时，数据的一致性可以保持，而当这个单元中的一部分操作失败，整个事务应全部视为错误，所有从起始点以后的操作应全部回退到开始状态。 

### 6.2 JDBC事务处理

- 数据一旦提交，就不可回滚。
- 数据什么时候意味着提交？

  - DDL操作一旦执行，就会自行提交 【set_autocommit = false 对DDL操作无效】
  - DML默认情况下一旦执行，就会自动提交
  - **关闭数据库连接，数据就会自动的提交。**如果多个操作，每个操作使用的是自己单独的连接，则无法保证事务。即同一个事务的多个操作必须在同一个连接下。
- **JDBC程序中为了让多个 SQL 语句作为一个事务执行：**

  - 调用 Connection 对象的 **setAutoCommit(false);** 以取消自动提交事务
  - 在所有的 SQL 语句都成功执行后，调用 **commit();** 方法提交事务
  - 在出现异常时，调用 **rollback();** 方法回滚事务

  > **若此时 Connection 没有被关闭，还可能被重复使用，则需要恢复其自动提交状态 setAutoCommit(true)。尤其是在使用数据库连接池技术时，连接是不会被关闭和销毁的，使用完了直接放到池中，如果我们修改了默认的自动提交成手动提交，当另一个人拿到之后就会以为还是自动提交就会出现问题，执行close()方法前，建议恢复自动提交状态。**

【案例：用户AA向用户BB转账100】

```java
public void testJDBCTransaction() {
	Connection conn = null;
	try {
		// 1.获取数据库连接
		conn = JDBCUtils.getConnection();
		// 2.开启事务
		conn.setAutoCommit(false);
		// 3.进行数据库操作
		String sql1 = "update user_table set balance = balance - 100 where user = ?";
		update(conn, sql1, "AA");

		// 模拟网络异常
		System.out.println(10 / 0);

		String sql2 = "update user_table set balance = balance + 100 where user = ?";
		update(conn, sql2, "BB");
		// 4.若没有异常，则提交事务
		conn.commit();
	} catch (Exception e) {
		e.printStackTrace();
		// 5.若有异常，则回滚事务
		try {
			conn.rollback();
		} catch (SQLException e1) {
			e1.printStackTrace();
		}
    } finally {
        try {
			//6.恢复每次DML操作的自动提交功能
			conn.setAutoCommit(true);
		} catch (SQLException e) {
			e.printStackTrace();
		}
        //7.关闭连接
		JDBCUtils.closeResource(conn, null, null); 
    }  
}

```

其中，对数据库操作的方法为：

```java
//使用事务以后的通用的增删改操作（version 2.0）
public void update(Connection conn ,String sql, Object... args) {
	PreparedStatement ps = null;
	try {
		// 1.获取PreparedStatement的实例 (或：预编译sql语句)
		ps = conn.prepareStatement(sql);
		// 2.填充占位符
		for (int i = 0; i < args.length; i++) {
			ps.setObject(i + 1, args[i]);
		}
		// 3.执行sql语句
		ps.execute();
	} catch (Exception e) {
		e.printStackTrace();
	} finally {
		// 4.关闭资源
		JDBCUtils.closeResource(null, ps);

	}
}
```



### 6.3 事务的ACID属性    

1. **原子性（Atomicity）**
    原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。 

2. **一致性（Consistency）**
    事务必须使数据库从一个一致性状态变换到另外一个一致性状态。

3. **隔离性（Isolation）**
    事务的隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。

4. **持久性（Durability）**
    持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响。

#### 6.3.1 数据库的并发问题

- 对于同时运行的多个事务, 当这些事务访问数据库中相同的数据时, 如果没有采取必要的隔离机制, 就会导致各种并发问题:
  - **脏读**: 对于两个事务 T1, T2, T1 读取了已经被 T2 更新但还**没有被提交**的字段。之后, 若 T2 回滚, T1读取的内容就是临时且无效的。
  - **不可重复读**: 对于两个事务T1, T2, T1 读取了一个字段, 然后 T2 **更新**了该字段。之后, T1再次读取同一个字段, 值就不同了。
  - **幻读**: 对于两个事务T1, T2, T1 从一个表中读取了一个字段, 然后 T2 在该表中**插入**了一些新的行。之后, 如果 T1 再次读取同一个表, 就会多出几行。
- **数据库事务的隔离性**: 数据库系统必须具有隔离并发运行各个事务的能力, 使它们不会相互影响, 避免各种并发问题。
- 一个事务与其他事务隔离的程度称为隔离级别。数据库规定了多种事务隔离级别, 不同隔离级别对应不同的干扰程度, **隔离级别越高, 数据一致性就越好, 但并发性越弱。**

> 不可重复读，比如我们在浏览器上看到了库存量，然后在后台加了几条库存，此时我们重新刷新一下浏览器发现库存量多了（刷新浏览器连接不会关闭，只有退出浏览器重新打开才会重新建立连接），这时就是我们说的不可重复读的情况，两次读的数据不一样。其实这样是可以理解的
>
> 幻读，打开浏览器本来有10条数据，后台插了两条，此时刷新变成了12条，像产生幻觉了一样。

#### 6.3.2 四种隔离级别

- 数据库提供的4种事务隔离级别：

  ![1555586275271](https://image.imxyu.cn/file/1555586275271.png)

- Oracle 支持的 2 种事务隔离级别：**READ COMMITED**, SERIALIZABLE。 Oracle 默认的事务隔离级别为: **READ COMMITED** 。


- Mysql 支持 4 种事务隔离级别。Mysql 默认的事务隔离级别为: **REPEATABLE READ。**

> 不同的事务隔离级别就是来解决上面数据库并发情况下所带来的问题
>
> 读未提交 -> 什么都没解决
>
> 读已提交 -> 解决脏读
>
> 可重复读 -> 解决脏读和不可重复读
>
> 串行化解决上面所有的问题
>
> 越向下数据库的一致性越好，当然效率也就越低


#### 6.3.3 在MySql中设置隔离级别

- 每启动一个 mysql 程序, 就会获得一个单独的数据库连接. 每个数据库连接都有一个全局变量 @@tx_isolation, 表示当前的事务隔离级别。

- 查看当前的隔离级别: 

  ```mysql
  SELECT @@tx_isolation;
  ```

- 设置当前 mySQL 连接的隔离级别:  

  ```mysql
  set  transaction isolation level read committed;
  ```

- 设置数据库系统的全局的隔离级别:

  ```mysql
  set global transaction isolation level read committed;
  ```

- 补充操作：

  - 创建mysql数据库用户：

    ```mysql
    create user tom identified by 'abc123';
    ```

  - 授予权限

    ```mysql
    #授予通过网络方式登录的tom用户，对所有库所有表的全部权限，密码设为abc123.
    grant all privileges on *.* to tom@'%'  identified by 'abc123'; 
    
     #给tom用户使用本地命令行方式，授予atguigudb这个库下的所有表的插删改查的权限。
    grant select,insert,delete,update on atguigudb.* to tom@localhost identified by 'abc123'; 
    
    ```


### java代码实践不同隔离级别问题

#### 读未提交

```java
// 模拟两个事务，一个查询表中的字段，另一个修改表中的字段
@Test
    public void tx1() throws Exception {
            Connection connection = JDBCUtils.getConnection();
            //设置数据库的隔离级别
            connection.setTransactionIsolation(Connection.TRANSACTION_READ_UNCOMMITTED);
            //获取数据库的隔离级别
            int transactionIsolation = connection.getTransactionIsolation();
            System.out.println(transactionIsolation);
            String sql="select user,password,balance from user_table where user = ? ";
            //开启手动提交事务
            connection.setAutoCommit(false);
            User cc = queryForTx(connection, User.class, sql, "CC");
            System.out.println(cc);

    }
    @Test
    public void tx2() throws Exception {
        String sql="update user_table set balance = 5000 where user = ?";
        Connection connection = JDBCUtils.getConnection();
        //开启手动提交事务
        connection.setAutoCommit(false);
        update(connection,sql,"CC");
        Thread.sleep(15000);
    }

    //通用的增、删、改操作，这里手动传入了conn ，而不是每次都获取新的conn, 是为了验证同一连接conn下的事务操作
    public void update(Connection conn,String sql,Object ... args) throws Exception {
        PreparedStatement ps = null;
            //2.获取PreparedStatement的实例 (或：预编译sql语句)
            ps = conn.prepareStatement(sql);
            //3.填充占位符 ，注意这里是i+1, java都是从1开始
            for(int i = 0;i < args.length;i++){
                ps.setObject(i + 1, args[i]);
            }
            //4.执行sql语句
            ps.execute();

    }

// 查询用的我们之前通用查询的那个方法
```

> 注意上面的tx2 和update（） 中我们没有提交事务，也没有关闭连接，所以事务是没有提交的状态

首先我们来运行tx1()，把数据库的隔离级别设置成TRANSACTION_READ_UNCOMMITTED 读未提交的状态

然后查询了一下CC 这个用户，对应的balance 是3000

![image-20220309113500547](https://image.imxyu.cn/file/image-20220309113500547.png)



接下来我们运行第二个，然后我们再用tx1（）方法来查询一下，因为之前的tx1 使用的连接没有关闭，下一次我们获取的还是这个连接，我们还是在同一个事务中

结果发现，虽然tx2 执行了修改，但是没有提交，此时通过tx1 中查询竟然得到了另一个事务中没有提交的数，这就是脏读带来的问题

![image-20220309113911287](https://image.imxyu.cn/file/image-20220309113911287.png)



等到tx2() 15秒结束之后，我们再通过tx1() 进行查询，数据回到了3000，也就是说这个数据并没有提交到数据库

![image-20220309114245497](https://image.imxyu.cn/file/image-20220309114245497.png)

#### 读已提交

接下来我们设置成读已提交

![image-20220309134531359](https://image.imxyu.cn/file/image-20220309134531359.png)

先执行tx1() 查询是3000，然后执行tx2() 将其设置成5000，并且还是没有提交，此时我们再执行tx1 ，查到的还是3000，

也就是说读已提交解决了在数据库高并发情况下访问数据库时的**脏读问题**

![image-20220309134448361](https://image.imxyu.cn/file/image-20220309134448361.png)



> 需要知道的一点就是我们可以在java代码中设置数据库的隔离级别
>
> 其实常用的设置成读已提交（只有提交的数据我们才可以读）就可以了，对于mysql默认的隔离级别为： 可重复读



## 第7章：DAO及相关实现类

- DAO：Data Access Object访问数据信息的类和接口，包括了对数据的CRUD（Create、Retrival、Update、Delete），而不包含任何业务相关的信息。有时也称作：BaseDAO
- 作用：为了实现功能的模块化，更有利于代码的维护和升级。
- 下面是尚硅谷JavaWeb阶段书城项目中DAO使用的体现：

![1566726681515](https://image.imxyu.cn/file/1566726681515.png)

- 层次结构：

![1566745811244](https://image.imxyu.cn/file/1566745811244.png)

> 通过basedao抽象类只是通过提供继承，可以提供一些通用的基础方法，而接口是为了实现规范化



- DAO：Data Access Object访问数据信息的类和接口，包括了对数据的CRUD，而不包含任何业务相关的信息。有时也称作：BaseDAO
- 作用：为了实现功能的模块化，更有利于代码的维护和升级

### BaseDao的实现

我们在原先的查询和更新的基础上 额外提供一个可以使用聚合函数查询（列数或者最大生日的）一个getValue（）这样一个方法

这个方法可以返回long 类型的整数，也可以返回Date类型，那么我们的返回值设成什么呢？ 

第一反应是object ，但是那样的话得到之后需要我们手动强制类型转换，所以我们可以设置成泛型，返回任意类型，也就是得到什么类型就转成什么类型

比object 的好处就是不需要我们进行强转了，object的话强转还可能出现问题

```java
public abstract class BaseDao {
    //事务通用查询
    public <T> List<T> getInstance(Connection conn, Class<T> clazz, String sql, Object... args) {
        List<T> list = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            ps = conn.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                ps.setObject(i + 1, args[i]);
            }
            rs = ps.executeQuery();
            ResultSetMetaData rsmd = rs.getMetaData();
            int columnCount = rsmd.getColumnCount();
            list = new ArrayList<>();
            while (rs.next()) {
                T t = clazz.newInstance();
                for (int i = 0; i < columnCount; i++) {
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    Object value = rs.getObject(i + 1);
                    //反射类所有字段，并且赋值
                    Field field = clazz.getDeclaredField(columnLabel);
                    field.setAccessible(true);
                    field.set(t, value);
                }
                list.add(t);
            }
        } catch (Exception throwables) {
            throwables.printStackTrace();
        } finally {
            JDBCUtils.closeResource(null, ps, rs);
        }
        return list;
    }

    //事务通用更新
    public <T> int getUpdate(Connection conn, String sql, Object... args) {
        List<T> list = null;
        PreparedStatement ps = null;
        int row = -1;
        try {
            ps = conn.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                ps.setObject(i + 1, args[i]);
            }
            row = ps.executeUpdate();
        } catch (Exception throwables) {
            throwables.printStackTrace();
        } finally {
            JDBCUtils.closeResource(null, ps);
            return row;
        }
    }

	//返回泛型（可以代表任意类型）比object 好
    public <T> T getValue(Connection conn, String sql, Object... args) {
        Object val = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            ps = conn.prepareStatement(sql);
            val = null;
            for (int i = 0; i < args.length; i++) {
                ps.setObject(i + 1, args[i]);
            }
            rs = ps.executeQuery();
            if (rs.next()) {
                val = rs.getObject(1);
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JDBCUtils.closeResource(null, ps, rs);
        }
        return (T) val;
    }
}
```

### Dao接口

```java

public interface CustomerDao {
    /**
     * 将cust对象添加到数据库之中
     * @param conn
     * @param cust
     */
    void insert(Connection conn, Customers cust);

    /**
     * 根据id号删除数据库中customer数据
     * @param conn
     * @param id
     */
    void deleteById(Connection conn, int id);

    /**
     * 根据id号修改数据库中customer数据
     * @param conn
     * @param cust
     * @param id
     */
    void updateById(Connection conn,Customers cust, int id);

    /**
     * 根据id号查詢数据库中customer数据
     * @param conn
     * @param id
     */
    Customers getCustomerById(Connection conn, int id);

    /**
     * 查询数据库customer表中所有的数据
     * @param conn
     */
    List<Customers> getAll(Connection conn);

    /**
     * 查询数据库customer表中所有的行数
     * @param conn
     */
    long getCount(Connection conn);

    /**
     * 查询数据库customer表中年龄最大的生日
     * @param conn
     */
    Date getMaxBirth(Connection conn);
}
```

### Dao接口实现类

```java

public class CustomerDaoImpl extends BaseDao implements CustomerDao{

    @Override
    public void insert(Connection conn, Customers cust) {
        String sql = "insert into customers(name,email,birth) values(?,?,?)";
        getUpdate(conn,sql,cust.getName(),cust.getEmail(),cust.getBirth());
    }

    @Override
    public void deleteById(Connection conn, int id) {
        String sql = "delete from customers where id = ?";
        getUpdate(conn,sql,id);
    }

    @Override
    public void updateById(Connection conn, Customers cust, int id) {
        String sql = "update customers set name = ?, email = ?, birth = ? where id = ?";
        getUpdate(conn,sql,cust.getName(),cust.getEmail(),cust.getBirth(),id);
    }

    @Override
    public Customers getCustomerById(Connection conn, int id) {
        String sql = "select name, email, birth from customers where id = ?";
        List<Customers> list = getInstance(conn, Customers.class, sql, id);
        return list.size() > 0 ? list.get(0) :null;
    }

    @Override
    public List<Customers> getAll(Connection conn) {
        String sql = "select name, email, birth from customers";
        List<Customers> list = getInstance(conn, Customers.class, sql);
        return list;
    }

    @Override
    public long getCount(Connection conn) {
        String sql = "select count(*) from customers";
        return  getValue(conn,sql);
    }
	
    @Override
    public Date getMaxBirth(Connection conn) {
        String sql = "select max(birth) from customers";
        return getValue(conn,sql);
    }
}
```

### 实现类的单元测试

Ctril + Shift + T 创建实现类的单元测试类

![image-20220309161043653](https://image.imxyu.cn/file/image-20220309161043653.png)

💡 如果弹出的窗口不是*Create new Test*，则说明，该类的父类中或者接口中存在@Test相关内容，删除即可

```jsx
public class CustomerDaoImplTest {

    CustomerDaoImpl custImpl = new CustomerDaoImpl();
    @Test
    public void insert() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConnection();
            Customers customer = new Customers(1,"jack","jack@163.com",new Date(543534543534L));
            custImpl.insert(conn,customer);
            System.out.println("添加成功");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(conn,null);
        }
    }

    @Test
    public void deleteById() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConnection();
            custImpl.deleteById(conn,19);
            System.out.println("删除成功");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(conn,null);
        }
    }

    @Test
    public void updateById() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConnection();
            Customers customer = new Customers(2,"jack.ma","jack@163.com",new Date(543534543534L));
            custImpl.updateById(conn,customer,20);
            System.out.println("更新成功");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(conn,null);
        }
    }

    @Test
    public void getCustomerById() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConnection();
            Customers cust = custImpl.getCustomerById(conn, 20);
            System.out.println(cust);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(conn,null);
        }
    }

    @Test
    public void getAll() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConnection();
            List<Customers> list = custImpl.getAll(conn);
            list.forEach(System.out::println);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(conn,null);
        }
    }

    @Test
    public void getCount() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConnection();
            long count = custImpl.getCount(conn);
            System.out.println(count);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(conn,null);
        }
    }

    @Test
    public void getMaxBirth() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConnection();
            Date birdth = custImpl.getMaxBirth(conn);
            System.out.println(birdth);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
}
```

BaseDao：实现对表的通用操作：通用增删改查

Dao接口: 定于对于某个表的的抽象方法

Dao接口实现类：实现接口中的抽象方法



### 优化方案

![image-20220309163353039](https://image.imxyu.cn/file/image-20220309163353039.png)



优化前：我们发现在BaseDao类中，需要传入具体的类型的Class对象，其实是没有必要的，因为我们对于CustomerDaoImpl 就是对Customer 的操作，所以这里可以省略的，那么我们看一下如何省？

```jsx
//未优化：需要传入clazz
public <T> List<T> getInstance(Connection conn, Class<T> clazz, String sql, Object... args) {
```

优化后：BaseDao类中，不需要传入Class对象

因为我们在进行类操作的时候就已经知道要操作哪个表了，因此我们需要在类上定义一个泛型

![image-20220309172027161](https://image.imxyu.cn/file/image-20220309172027161.png)

而对于其他类要操作指定的表，就要传入要操作的哪个类型的。此时我们就知道要操作的哪个类型的了

![image-20220309172034588](https://image.imxyu.cn/file/image-20220309172034588.png)

```
private Class<T> clazz 
```

而对于这个属性，因为这是一个通用的属性，所有继承于BaseDao的都需要有这个属性，所以我们抽到外面的属性中，这样其他子类就都有了这个属性。另外我们需要保证在这个BaseDao类的方法被对象调用的时候有值就行，那么我们可以用什么手段呢？



一种是代码块的方式，或者构造函数的形式 。 这样在new 子类对象的时候，会默认有一个super() ,这个方法就会把父类的代码块和构造器给加载了

首先介绍一种天真的、错误的：

```java
private Class<T> clazz = T; // 首先这种肯定是错误的，没有这么用的
```

现在我们要解决的一个问题是，我们需要根据不同的子类传入的类型，在运行的时候动态获取到这个泛型类型 T ， 就可以通过下面的方法反射得到

![image-20220309170356003](https://image.imxyu.cn/file/image-20220309170356003.png)

> 这里解释一下，因为所有的类都需要这个参数，因此把这个方法和参数放到父类中，而当我们new 子类，调用子类方法的时候，下面这个this  就是子类的对象，然后获取当前对象的带泛型的父类的类型，就可以了
>
> 比如说下面这个CustomerDaoImpl dao=new CustomerDaoImpl() ; 
>
> 当我们创建完子类对象的时候，在构造器中会通过super（） 加载父类的构造函数和代码块，此时在父类中会获取到当前对象的带泛型的父类，然后获取到泛型父类上的泛型类型， 这里就是Customer 这个类类型，这样我们就得到了这个泛型类型变量

![image-20220309170637571](https://image.imxyu.cn/file/image-20220309170637571.png)

而对于我们之前说的getValue 这个方法就是一个普通的泛型方法。

![image-20220309171052509](https://image.imxyu.cn/file/image-20220309171052509.png)



### 优化总结

首先我们要去除掉这个Class类型参数，因为我们对于其他表都已经明确知道我们要操作哪个表了，

1、此时我们就要传入我们要具体操作的表类型，所以在类上定义这个类型，在子类继承的时候指定要操作的表类型

2、接下来就要在运行时根据不同的子类传入来的这个类型动态的获取到这个类型，这里就用到了反射。这里需要重点理解一下把这个代码块放到父类的加载过程



## 第8章：数据库连接池

### 8.1 JDBC数据库连接池的必要性

- 在使用开发基于数据库的web程序时，传统的模式基本是按以下步骤：　　
  - **在主程序（如servlet、beans）中建立数据库连接**
  - **进行sql操作**
  - **断开数据库连接**

- 这种模式开发，存在的问题:
  - 普通的JDBC数据库连接使用 DriverManager 来获取，每次向数据库建立连接的时候都要将 Connection 加载到内存中，再验证用户名和密码(得花费0.05s～1s的时间)。需要数据库连接的时候，就向数据库要求一个，执行完成后再断开连接。这样的方式将会消耗大量的资源和时间。**数据库的连接资源并没有得到很好的重复利用。**若同时有几百人甚至几千人在线，频繁的进行数据库连接操作将占用很多的系统资源，严重的甚至会造成服务器的崩溃。
  - **对于每一次数据库连接，使用完后都得断开。**否则，如果程序出现异常而未能关闭，将会导致数据库系统中的内存泄漏，最终将导致重启数据库。（回忆：何为Java的内存泄漏？ 就是有对象不能被回收，对象一直存在。）
  - **这种开发不能控制被创建的连接对象数**，系统资源会被毫无顾及的分配出去，如连接过多，也可能导致内存泄漏，服务器崩溃。 

### 8.2 数据库连接池技术

- 为解决传统开发中的数据库连接问题，可以采用数据库连接池技术。
- **数据库连接池的基本思想**：就是为数据库连接建立一个“缓冲池”。预先在缓冲池中放入一定数量的连接，当需要建立数据库连接时，只需从“缓冲池”中取出一个，使用完毕之后再放回去。

- **数据库连接池**负责分配、管理和释放数据库连接，它**允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个**。
- 数据库连接池在初始化时将创建一定数量的数据库连接放到连接池中，这些数据库连接的数量是由**最小数据库连接数来设定**的。无论这些数据库连接是否被使用，连接池都将一直保证至少拥有这么多的连接数量。连接池的**最大数据库连接数量**限定了这个连接池能占有的最大连接数，当应用程序向连接池请求的连接数超过最大连接数量时，这些请求将被加入到等待队列中。

![1555593464033](https://image.imxyu.cn/file/1555593464033.png)

- **工作原理：**

![1555593598606](https://image.imxyu.cn/file/1555593598606.png)

- **数据库连接池技术的优点**

  **1. 资源重用**

  由于数据库连接得以重用，避免了频繁创建，释放连接引起的大量性能开销。在减少系统消耗的基础上，另一方面也增加了系统运行环境的平稳性。

  **2. 更快的系统反应速度**

  数据库连接池在初始化过程中，往往已经创建了若干数据库连接置于连接池中备用。此时连接的初始化工作均已完成。对于业务请求处理而言，直接利用现有可用连接，避免了数据库连接初始化和释放过程的时间开销，从而减少了系统的响应时间

  **3. 新的资源分配手段**

  对于多应用共享同一数据库的系统而言，可在应用层通过数据库连接池的配置，实现某一应用最大可用数据库连接数的限制，避免某一应用独占所有的数据库资源

  **4. 统一的连接管理，避免数据库连接泄漏**

  在较为完善的数据库连接池实现中，可根据预先的占用超时设定，强制回收被占用连接，从而避免了常规数据库连接操作中可能出现的资源泄露


### 8.3 多种开源的数据库连接池

- JDBC 的数据库连接池使用 javax.sql.DataSource 来表示，DataSource 只是一个接口，该接口通常由服务器(Weblogic, WebSphere, Tomcat)提供实现，也有一些开源组织提供实现：
  - **DBCP** 是Apache提供的数据库连接池。tomcat 服务器自带dbcp数据库连接池。**速度相对c3p0较快**，但因自身存在BUG，Hibernate3已不再提供支持。
  - **C3P0** 是一个开源组织提供的一个数据库连接池，**速度相对较慢，稳定性还可以。**hibernate官方推荐使用
  - **Proxool** 是sourceforge下的一个开源项目数据库连接池，有监控连接池状态的功能，**稳定性较c3p0差一点**
  - **BoneCP** 是一个开源组织提供的数据库连接池，速度快
  - **Druid** 是阿里提供的数据库连接池，据说是集DBCP 、C3P0 、Proxool 优点于一身的数据库连接池，但是速度不确定是否有BoneCP快
- DataSource 通常被称为数据源，它包含连接池和连接池管理两个部分，习惯上也经常把 DataSource 称为连接池
- **DataSource用来取代DriverManager来获取Connection，获取速度快，同时可以大幅度提高数据库访问速度。**
- 特别注意：
  - 数据源和数据库连接不同，数据源无需创建多个，它是产生数据库连接的工厂，因此**整个应用只需要一个数据源即可。**
  - 当数据库访问结束后，程序还是像以前一样关闭数据库连接：conn.close(); 但conn.close()并没有关闭数据库的物理连接，它仅仅把数据库连接释放，归还给了数据库连接池。

>  DataSource 只是一个接口，被各个厂商去实现，就像我们之前说的driver 驱动一样，被各个数据库厂商去实现，我们不需要关注每一个具体的细节

#### 8.3.1 C3P0数据库连接池

- 获取连接方式一

```java
//使用C3P0数据库连接池的方式，获取数据库的连接：不推荐
public static Connection getConnection1() throws Exception{
	ComboPooledDataSource cpds = new ComboPooledDataSource();
	cpds.setDriverClass("com.mysql.jdbc.Driver"); 
	cpds.setJdbcUrl("jdbc:mysql://localhost:3306/test");
	cpds.setUser("root");
	cpds.setPassword("abc123");
		
//	cpds.setMaxPoolSize(100);
	
	Connection conn = cpds.getConnection();
	return conn;
}
```

- 获取连接方式二

```java
//使用C3P0数据库连接池的配置文件方式，获取数据库的连接：推荐
private static DataSource cpds = new ComboPooledDataSource("helloc3p0");
public static Connection getConnection2() throws SQLException{
	Connection conn = cpds.getConnection();
	return conn;
}
```

> 这里我们把连接池抽出去，不需要每次获取连接时候都创建一个，只需要用一个就行

其中，src下的配置文件为：【c3p0-config.xml】

```xml
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
	<named-config name="helloc3p0">
		<!-- 获取连接的4个基本信息 -->
		<property name="user">root</property>
		<property name="password">abc123</property>
		<property name="jdbcUrl">jdbc:mysql:///test</property>
		<property name="driverClass">com.mysql.jdbc.Driver</property>
		
		<!-- 涉及到数据库连接池的管理的相关属性的设置 -->
		<!-- 若数据库中连接数不足时, 一次向数据库服务器申请多少个连接 -->
		<property name="acquireIncrement">5</property>
		<!-- 初始化数据库连接池时连接的数量 -->
		<property name="initialPoolSize">5</property>
		<!-- 数据库连接池中的最小的数据库连接数 -->
		<property name="minPoolSize">5</property>
		<!-- 数据库连接池中的最大的数据库连接数 -->
		<property name="maxPoolSize">10</property>
		<!-- C3P0 数据库连接池可以维护的 Statement 的个数 -->
		<property name="maxStatements">20</property>
		<!-- 每个连接同时可以使用的 Statement 对象的个数 -->
		<property name="maxStatementsPerConnection">5</property>

	</named-config>
</c3p0-config>
```

> 使用数据库配置文件的方式， 会自动去src下找 c3p0-config.xml这个名字的配置文件，获取里面对应的值，
>
> 注意我们new ComboPooledDataSource("helloc3p0"); 参数里面传入的是named-config 配置名字，而不是 c3p0-config.xml 这个文件名

#### 8.3.2 DBCP数据库连接池

- DBCP 是 Apache 软件基金组织下的开源连接池实现，该连接池依赖该组织下的另一个开源系统：Common-pool。如需使用该连接池实现，应在系统中增加如下两个 jar 文件：
  - Commons-dbcp.jar：连接池的实现
  - Commons-pool.jar：连接池实现的依赖库
- **Tomcat 的连接池正是采用该连接池来实现的。**该数据库连接池既可以与应用服务器整合使用，也可由应用程序独立使用。
- 数据源和数据库连接不同，数据源无需创建多个，它是产生数据库连接的工厂，因此整个应用只需要一个数据源即可。
- 当数据库访问结束后，程序还是像以前一样关闭数据库连接：conn.close(); 但上面的代码并没有关闭数据库的物理连接，它仅仅把数据库连接释放，归还给了数据库连接池。
- 配置属性说明

| 属性                       | 默认值 | 说明                                                         |
| -------------------------- | ------ | ------------------------------------------------------------ |
| initialSize                | 0      | 连接池启动时创建的初始化连接数量                             |
| maxActive                  | 8      | 连接池中可同时连接的最大的连接数                             |
| maxIdle                    | 8      | 连接池中最大的空闲的连接数，超过的空闲连接将被释放，如果设置为负数表示不限制 |
| minIdle                    | 0      | 连接池中最小的空闲的连接数，低于这个数量会被创建新的连接。该参数越接近maxIdle，性能越好，因为连接的创建和销毁，都是需要消耗资源的；但是不能太大。 |
| maxWait                    | 无限制 | 最大等待时间，当没有可用连接时，连接池等待连接释放的最大时间，超过该时间限制会抛出异常，如果设置-1表示无限等待 |
| poolPreparedStatements     | false  | 开启池的Statement是否prepared                                |
| maxOpenPreparedStatements  | 无限制 | 开启池的prepared 后的同时最大连接数                          |
| minEvictableIdleTimeMillis |        | 连接池中连接，在时间段内一直空闲， 被逐出连接池的时间        |
| removeAbandonedTimeout     | 300    | 超过时间限制，回收没有用(废弃)的连接                         |
| removeAbandoned            | false  | 超过removeAbandonedTimeout时间后，是否进 行没用连接（废弃）的回收 |



- 获取连接方式一：

```java
public static Connection getConnection3() throws Exception {
	BasicDataSource source = new BasicDataSource();
		
	source.setDriverClassName("com.mysql.jdbc.Driver");
	source.setUrl("jdbc:mysql:///test");
	source.setUsername("root");
	source.setPassword("abc123");
		
	//
	source.setInitialSize(10);
		
	Connection conn = source.getConnection();
	return conn;
}
```

- 获取连接方式二：

```java
//使用dbcp数据库连接池的配置文件方式，获取数据库的连接：推荐
private static DataSource source = null;
static{
	try {
		Properties pros = new Properties();
		
		InputStream is = DBCPTest.class.getClassLoader().getResourceAsStream("dbcp.properties");
			
		pros.load(is);
		//根据提供的BasicDataSourceFactory创建对应的DataSource对象
		source = BasicDataSourceFactory.createDataSource(pros);
	} catch (Exception e) {
		e.printStackTrace();
	}
		
}
public static Connection getConnection4() throws Exception {
		
	Connection conn = source.getConnection();
	
	return conn;
}
```

> 同样这里我们也把这个连接池抽取出去，但是这里需要进行读取文件的操作，不能当为属性，
>
> 此时我们可以利用静态代码块的方式，让这段代码也就执行一次，用一个数据库连接池即可

其中，src下的配置文件为：【dbcp.properties】

```properties
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?rewriteBatchedStatements=true&useServerPrepStmts=false
username=root
password=abc123

initialSize=10
#...
```



#### 8.3.3 Druid（德鲁伊）数据库连接池

Druid是阿里巴巴开源平台上一个数据库连接池实现，它结合了C3P0、DBCP、Proxool等DB池的优点，同时加入了日志监控，可以很好的监控DB池连接和SQL的执行情况，可以说是针对监控而生的DB连接池，**可以说是目前最好的连接池之一。**

```java
package com.atguigu.druid;

import java.sql.Connection;
import java.util.Properties;

import javax.sql.DataSource;

import com.alibaba.druid.pool.DruidDataSourceFactory;

public class TestDruid {
	public static void main(String[] args) throws Exception {
		Properties pro = new Properties();		 pro.load(TestDruid.class.getClassLoader().getResourceAsStream("druid.properties"));
        //使用提供的工厂加载配置文件获取datasource
		DataSource ds = DruidDataSourceFactory.createDataSource(pro);
		Connection conn = ds.getConnection();
		System.out.println(conn);
	}
}

```

其中，src下的配置文件为：【druid.properties】

```java
url=jdbc:mysql://localhost:3306/test?rewriteBatchedStatements=true
username=root
password=123456
driverClassName=com.mysql.jdbc.Driver

initialSize=10
maxActive=20
maxWait=1000
filters=wall
```

- 详细配置参数：

| **配置**                      | **缺省** | **说明**                                                     |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| name                          |          | 配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。   如果没有配置，将会生成一个名字，格式是：”DataSource-” +   System.identityHashCode(this) |
| url                           |          | 连接数据库的url，不同数据库不一样。例如：mysql :   jdbc:mysql://10.20.153.104:3306/druid2      oracle :   jdbc:oracle:thin:@10.20.149.85:1521:ocnauto |
| username                      |          | 连接数据库的用户名                                           |
| password                      |          | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。详细看这里：<https://github.com/alibaba/druid/wiki/%E4%BD%BF%E7%94%A8ConfigFilter> |
| driverClassName               |          | 根据url自动识别   这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName(建议配置下) |
| initialSize                   | 0        | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive                     | 8        | 最大连接池数量                                               |
| maxIdle                       | 8        | 已经不再使用，配置了也没效果                                 |
| minIdle                       |          | 最小连接池数量                                               |
| maxWait                       |          | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| poolPreparedStatements        | false    | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 |
| maxOpenPreparedStatements     | -1       | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 |
| validationQuery               |          | 用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会其作用。 |
| testOnBorrow                  | true     | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn                  | false    | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 |
| testWhileIdle                 | false    | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。 |
| timeBetweenEvictionRunsMillis |          | 有两个含义： 1)Destroy线程会检测连接的间隔时间2)testWhileIdle的判断依据，详细看testWhileIdle属性的说明 |
| numTestsPerEvictionRun        |          | 不再使用，一个DruidDataSource只支持一个EvictionRun           |
| minEvictableIdleTimeMillis    |          |                                                              |
| connectionInitSqls            |          | 物理连接初始化的时候执行的sql                                |
| exceptionSorter               |          | 根据dbType自动识别   当数据库抛出一些不可恢复的异常时，抛弃连接 |
| filters                       |          | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：   监控统计用的filter:stat日志用的filter:log4j防御sql注入的filter:wall |
| proxyFilters                  |          | 类型是List，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系 |



## 第9章：Apache-DBUtils实现CRUD操作

### 9.1 Apache-DBUtils简介

- commons-dbutils 是 Apache 组织提供的一个开源 JDBC工具类库，它是对JDBC的简单封装，学习成本极低，并且使用dbutils能极大简化jdbc编码的工作量，同时也不会影响程序的性能。

- API介绍：
  - org.apache.commons.dbutils.QueryRunner
  - org.apache.commons.dbutils.ResultSetHandler
  - 工具类：org.apache.commons.dbutils.DbUtils   
- API包说明：

![1555595163263](https://image.imxyu.cn/file/1555595163263.png)

![1555595198644](https://image.imxyu.cn/file/1555595198644.png)





### 9.2 主要API的使用

#### 9.2.1 DbUtils

- DbUtils ：提供如关闭连接、装载JDBC驱动程序等常规工作的工具类，里面的所有方法都是静态的。主要方法如下：
  - **public static void close(…) throws java.sql.SQLException**：　DbUtils类提供了三个重载的关闭方法。这些方法检查所提供的参数是不是NULL，如果不是的话，它们就关闭Connection、Statement和ResultSet。
  - public static void closeQuietly(…): 这一类方法不仅能在Connection、Statement和ResultSet为NULL情况下避免关闭，还能隐藏一些在程序中抛出的SQLEeception。
  - public static void commitAndClose(Connection conn)throws SQLException： 用来提交连接的事务，然后关闭连接
  - public static void commitAndCloseQuietly(Connection conn)： 用来提交连接，然后关闭连接，并且在关闭连接时不抛出SQL异常。 
  - public static void rollback(Connection conn)throws SQLException：允许conn为null，因为方法内部做了判断
  - public static void rollbackAndClose(Connection conn)throws SQLException
  - rollbackAndCloseQuietly(Connection)
  - public static boolean loadDriver(java.lang.String driverClassName)：这一方装载并注册JDBC驱动程序，如果成功就返回true。使用该方法，你不需要捕捉这个异常ClassNotFoundException。

#### 9.2.2 QueryRunner类

- **该类简单化了SQL查询，它与ResultSetHandler组合在一起使用可以完成大部分的数据库操作，能够大大减少编码量。**

- QueryRunner类提供了两个构造器：
  - 默认的构造器
  - 需要一个 javax.sql.DataSource 来作参数的构造器

- QueryRunner类的主要方法：
  - **更新**
    - public int update(Connection conn, String sql, Object... params) throws SQLException:用来执行一个更新（插入、更新或删除）操作。
    -  ......
  - **插入**
    - public <T> T insert(Connection conn,String sql,ResultSetHandler<T> rsh, Object... params) throws SQLException：只支持INSERT语句，其中 rsh - The handler used to create the result object from the ResultSet of auto-generated keys.  返回值: An object generated by the handler.即自动生成的键值
    - ....
  - **批处理**
    - public int[] batch(Connection conn,String sql,Object[][] params)throws SQLException： INSERT, UPDATE, or DELETE语句
    - public <T> T insertBatch(Connection conn,String sql,ResultSetHandler<T> rsh,Object[][] params)throws SQLException：只支持INSERT语句
    - .....
  - **查询**
    - public Object query(Connection conn, String sql, ResultSetHandler rsh,Object... params) throws SQLException：执行一个查询操作，在这个查询中，对象数组中的每个元素值被用来作为查询语句的置换参数。该方法会自行处理 PreparedStatement 和 ResultSet 的创建和关闭。
    - ...... 

- 测试

```java
// 测试添加
@Test
public void testInsert() throws Exception {
	QueryRunner runner = new QueryRunner();
	Connection conn = JDBCUtils.getConnection3();
	String sql = "insert into customers(name,email,birth)values(?,?,?)";
	int count = runner.update(conn, sql, "何成飞", "he@qq.com", "1992-09-08");

	System.out.println("添加了" + count + "条记录");
		
	JDBCUtils.closeResource(conn, null);

}
```

```java
// 测试删除
@Test
public void testDelete() throws Exception {
	QueryRunner runner = new QueryRunner();
	Connection conn = JDBCUtils.getConnection3();
	String sql = "delete from customers where id < ?";
	int count = runner.update(conn, sql,3);

	System.out.println("删除了" + count + "条记录");
		
	JDBCUtils.closeResource(conn, null);

}
```



#### 9.2.3 ResultSetHandler接口及实现类

- 该接口用于处理 java.sql.ResultSet，将数据按要求转换为另一种形式。

- ResultSetHandler 接口提供了一个单独的方法：Object handle (java.sql.ResultSet .rs)。

- 接口的主要实现类：

  - ArrayHandler：把结果集中的第一行数据转成对象数组。
  - ArrayListHandler：把结果集中的每一行数据都转成一个数组，再存放到List中。
  - **BeanHandler：**将结果集中的第一行数据封装到一个对应的JavaBean实例中。
  - **BeanListHandler：**将结果集中的每一行数据都封装到一个对应的JavaBean实例中，存放到List里。
  - ColumnListHandler：将结果集中某一列的数据存放到List中。
  - KeyedHandler(name)：将结果集中的每一行数据都封装到一个Map里，再把这些map再存到一个map里，其key为指定的key。
  - **MapHandler：**将结果集中的第一行数据封装到一个Map里，key是列名，value就是对应的值。
  - **MapListHandler：**将结果集中的每一行数据都封装到一个Map里，然后再存放到List
  - **ScalarHandler：**查询单个值对象

    

- 测试

```java
/*
 * 测试查询:查询一条记录
 * 
 * 使用ResultSetHandler的实现类：BeanHandler
 */
@Test
public void testQueryInstance() throws Exception{
	QueryRunner runner = new QueryRunner();

	Connection conn = JDBCUtils.getConnection3();
		
	String sql = "select id,name,email,birth from customers where id = ?";
		
	//
	BeanHandler<Customer> handler = new BeanHandler<>(Customer.class);
	Customer customer = runner.query(conn, sql, handler, 23);
	System.out.println(customer);	
	JDBCUtils.closeResource(conn, null);
}
```

```java
/*
 * 测试查询:查询多条记录构成的集合
 * 
 * 使用ResultSetHandler的实现类：BeanListHandler
 */
@Test
public void testQueryList() throws Exception{
	QueryRunner runner = new QueryRunner();

	Connection conn = JDBCUtils.getConnection3();
		
	String sql = "select id,name,email,birth from customers where id < ?";
		
	//
	BeanListHandler<Customer> handler = new BeanListHandler<>(Customer.class);
	List<Customer> list = runner.query(conn, sql, handler, 23);
	list.forEach(System.out::println);
		
	JDBCUtils.closeResource(conn, null);
}
```

```java
/*
 * 自定义ResultSetHandler的实现类
 */
@Test
public void testQueryInstance1() throws Exception{
	QueryRunner runner = new QueryRunner();

	Connection conn = JDBCUtils.getConnection3();
		
	String sql = "select id,name,email,birth from customers where id = ?";
		
	ResultSetHandler<Customer> handler = new ResultSetHandler<Customer>() {

		@Override
		public Customer handle(ResultSet rs) throws SQLException {
			System.out.println("handle");
//			return new Customer(1,"Tom","tom@126.com",new Date(123323432L));
				
			if(rs.next()){
				int id = rs.getInt("id");
				String name = rs.getString("name");
				String email = rs.getString("email");
				Date birth = rs.getDate("birth");
					
				return new Customer(id, name, email, birth);
			}
			return null;
				
		}
	};
		
	Customer customer = runner.query(conn, sql, handler, 23);
		
	System.out.println(customer);
		
	JDBCUtils.closeResource(conn, null);
}
```

```java
/*
 * 如何查询类似于最大的，最小的，平均的，总和，个数相关的数据，
 * 使用ScalarHandler
 * 
 */
@Test
public void testQueryValue() throws Exception{
	QueryRunner runner = new QueryRunner();

	Connection conn = JDBCUtils.getConnection3();
		
	//测试一：
//	String sql = "select count(*) from customers where id < ?";
//	ScalarHandler handler = new ScalarHandler();
//	long count = (long) runner.query(conn, sql, handler, 20);
//	System.out.println(count);
		
	//测试二：
	String sql = "select max(birth) from customers";
	ScalarHandler handler = new ScalarHandler();
	Date birth = (Date) runner.query(conn, sql, handler);
	System.out.println(birth);
		
	JDBCUtils.closeResource(conn, null);
}
```

##### 自定义Handler

```java
@Test
    //自定义resultHandler
    public  void test1() throws SQLException {
        //Druid连接池
        Connection conn = source.getConnection();
        //DBUtills
        QueryRunner runner = new QueryRunner();

        //查询单条结果
        String sql = "SELECT * FROM customers where id = 1";
        ResultSetHandler<Customer> selfDefineHandler = new ResultSetHandler<Customer>() {
            @Override
            public Customer handle(ResultSet resultSet) throws SQLException {
                ResultSetMetaData rsmd = resultSet.getMetaData();
                int columnCount = rsmd.getColumnCount();
                if(resultSet.next()){
                    Customer cust = new Customer();
                    for (int i = 0; i < columnCount; i++) {
                        String columnLabel = rsmd.getColumnLabel(i + 1);
                        Object value = resultSet.getObject(i + 1);
                        try {
                            Field field = Customer.class.getDeclaredField(columnLabel);
                            field.setAccessible(true);
                            field.set(cust,value);
												catch (Exception e) {
                            e.printStackTrace();
                        }
                        finally {
                            if(resultSet!=null)
                                resultSet.close();
                        }      
                    }
                    return cust;
                }
                return null;
            }
        };
        Customer query = runner.query(conn, sql, selfDefineHandler);
        System.out.println(query);

        //释放连接到资源池
        JDBCUtills.closeResource(conn,null);
    }
```

>  自定义ResultSetHandler接口的实现类，实现Handler方法 核心handle方法就是处理ResultSet 结果集，自定义返回类型

### 资源关闭

```java
		public void test1(Connection con, Statement st, ResultSet rs){
        //方式1
        try {
            DbUtils.close(con);
            DbUtils.close(st);
            DbUtils.close(rs);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        //方式2
        DbUtils.closeQuietly(con);
        DbUtils.closeQuietly(st);
        DbUtils.closeQuietly(rs);
    }
```

💡 Dbutills也提供资源关闭的方法，源码和自己写的是一样的



> **当我们使用了DBUtils就可以简化我们之前的BaseDao中的代码了，直接使用QueryRunner 已经提供好的这些方法就可以了，其实底层源码就是我们写的那些， 只不过多了一些细节的处理**

## JDBC总结



经过系统的学习JDBC核心技术，以下为个人总结的JDBC核心知识点，总体概括。

**概念理解**

- **公共接口**：JDBC就是SUM公司定义的一系列接口和标准，通用的SQL数据库存取和操作接口
- 驱动文件：如果想要融入Java的开发环境，那么各大数据库厂商（Mysql，Oracle，Sqlserver等）就需要实现公共接口，不同数据库厂商对接口的实现类，生成了各自的Jar包文件也被称为驱动文件。

 💡 SUN公司对数据库的操作设定标准，统一了方法名，参数类型，参数个数，返回类型，所有想要加入这个环境的数据库厂商，都要去实现这些接口。 那么我们就可以只使用公共的接口去操作数据库，而不用了解具体的接口实现类（也即是各个驱动）的技术实现细节



**环境准备**

- 加载驱动：将数据库厂商提供的驱动文件，加载到项目之中，而从能够调用公共接口的实现方法
- 获取连接：五种数据库连接方式，都可以对数据库进行连接，不断升级，提升了连接的效率，更加便捷。
- JDBCUtils：可以将数据库的连接与关闭单独封装到一个工具类之中，方便调用

**CURD**

- Statement：Sum公司提供的执行SQL语句的类，缺点是容易注入，拼串，效率低，无法操作Blob数据，无法批量执行SQL
- PrepareStatement：替代Statement的类，解决以上容易出现的各种缺陷，并且效率更高，特别是批量执行方面

**事务**

- 调用 Connection 对象的 **setAutoCommit(false);** 以取消自动提交事务
- 在所有的 SQL 语句都成功执行后，调用 **commit();** 方法提交事务
- 在出现异常时，调用 **rollback();** 方法回滚事务
- 若此时 Connection 没有被关闭，还可能被重复使用，则需要恢复其自动提交状态 setAutoCommit(true)。

💡 本质就是Java对MySQL事务的模拟，所以只要MySQL事务章节，学好了，就没问题了。

```sql
#1关闭自动提交事务
SET autocommit = 0;
#2开始事务
START TRANSACTION;

#3 编写一组事务的语句【select、insert、update、delete】
语句1
语句2
...
语句3

#4结束事务
commit; #提交
rollback; #回滚
```

**Dao**

对数据库的CURD，不包含具体的业务信息，优点是实现了功能的模块化，有利于代码维护和升级

- Dao层：对数据库通用的基础操作
- 表接口：定义某张表具有哪些数据库的操作，指定标准和规范
- 表接口实现类：对表接口进行功能的实现，调用Dao层的通用方法进行实现
- Bean层：方便进行数据库的存储，根据ORM映射思想，将数据库中表的结构，定义为Bean文件类

**数据库连接池**

每次数据库请求连接，关闭连接都很耗费时间，所以出现了数据库连接池技术，一次性开辟指定连接，每次请求都从连接池中获取，释放连接，实际上是释放到数据库连接池，而不是将连接关闭，实现对数据库连接的管理以及提升运行效率。

💡 数据库连接池技术出现后，基本替换了传统的连接方式，数据库连接池中比较常用的是阿里提供的Druid德鲁伊连接池



**Apache-DBUtils**

使用PreparedStatement对CURD进行封装，底层一致，对比自己写的健壮性更好一点 由于对于数据的增删改查操作非常频繁，所以官方将这些比较通用的方法封装成了一个工具类，供我们使用，底层方法其实和例子中的是一致的，健壮性更好些而已。

 💡 使用数据库连接池技术进行数据库连接。 使用DBUtils中的通用方法进行增删改查。 基本不手写JDBC对数据库进行操作，但是要明白底层的源码。 知其然，不知其所以然是不行的。

