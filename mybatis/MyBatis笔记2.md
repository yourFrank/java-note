视频地址：https://www.bilibili.com/video/BV1VP4y1c7j7?from=search&seid=1895879482480987332&spm_id_from=333.337.0.0

# 自定义映射resultMap

## resultMap处理字段和属性的映射关系

- resultMap：设置自定义映射  
 - 属性：  
   - id：表示自定义映射的唯一标识，不能重复
    - type：查询的数据要映射的实体类的类型  
   - 子标签：  
      - id：设置主键的映射关系  
     - result：设置普通字段的映射关系  
     - 子标签属性：  
      - property：设置映射关系中实体类中的属性名  
        - column：设置映射关系中表中的字段名
- 若字段名和实体类中的属性名不一致，则可以通过resultMap设置自定义映射，即使字段名和属性名一致的属性也要映射，也就是全部属性都要列出来

```xml
<resultMap id="empResultMap" type="Emp">
	<id property="eid" column="eid"></id>
	<result property="empName" column="emp_name"></result>
	<result property="age" column="age"></result>
	<result property="sex" column="sex"></result>
	<result property="email" column="email"></result>
</resultMap>
<!--List<Emp> getAllEmp();-->
<select id="getAllEmp" resultMap="empResultMap">
	select * from t_emp
</select>
```



如果还是想使用resultType的话

- 若字段名和实体类中的属性名不一致，但是字段名符合数据库的规则（使用_），实体类中的属性名符合Java的规则（使用驼峰）。此时也可通过以下两种方式处理字段名和实体类中的属性的映射关系  

~~~xml
1. 可以通过为字段起别名的方式，保证和实体类中的属性名保持一致  

	<!--List<Emp> getAllEmp();-->
	<select id="getAllEmp" resultType="Emp">
		select eid,emp_name empName,age,sex,email from t_emp
	</select>
	```
2. 可以在MyBatis的核心配置文件中的`setting`标签中，设置一个全局配置信息mapUnderscoreToCamelCase，可以在查询表中数据时，自动将_类型的字段名转换为驼峰，例如：字段名user_name，设置了mapUnderscoreToCamelCase，此时字段名就会转换为userName。 这个默认为false,也就是不开启的，需要我们手动开启

<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>

~~~

## 多对一映射处理

>查询员工信息以及员工所对应的部门信息

```java
public class Emp {  
	private Integer eid;  
	private String empName;  
	private Integer age;  
	private String sex;  
	private String email;  
	private Dept dept; //在多的一方设置部门对象
	//...构造器、get、set方法等
}
```

### 方法1：级联方式处理映射关系

因为我们查询出来的部门的信息，在emp 对象中肯定是没有这两个字段的，因为我们的emp中的是dept对象。所以这种情况我们只能使用resultMap来自定义映射关系

![image-20220310164617564](https://image.imxyu.cn/file/image-20220310164617564.png)

```xml
<resultMap id="empAndDeptResultMapOne" type="Emp">
	<id property="eid" column="eid"></id>
	<result property="empName" column="emp_name"></result>
	<result property="age" column="age"></result>
	<result property="sex" column="sex"></result>
	<result property="email" column="email"></result>
	<result property="dept.did" column="did"></result>
	<result property="dept.deptName" column="dept_name"></result>
</resultMap>
<!--Emp getEmpAndDept(@Param("eid")Integer eid);-->
<select id="getEmpAndDept" resultMap="empAndDeptResultMapOne">
	select * from t_emp left join t_dept on t_emp.eid = t_dept.did where t_emp.eid = #{eid}
</select>
```

> <result property="dept.did" column="did"></result>  就是把查到的did列，赋给dept属性对象的did 

### 方法2：使用association处理映射关系

- association：专门用来处理多对一的映射关系
- property：需要处理多对的映射关系的属性名
- javaType：该属性的类型，这里可以使用别名直接使用Dept作为类型

```xml
<resultMap id="empAndDeptResultMapTwo" type="Emp">
	<id property="eid" column="eid"></id>
	<result property="empName" column="emp_name"></result>
	<result property="age" column="age"></result>
	<result property="sex" column="sex"></result>
	<result property="email" column="email"></result>
	<association property="dept" javaType="Dept">
		<id property="did" column="did"></id>
		<result property="deptName" column="dept_name"></result>
	</association>
</resultMap>
<!--Emp getEmpAndDept(@Param("eid")Integer eid);-->
<select id="getEmpAndDept" resultMap="empAndDeptResultMapTwo">
	select * from t_emp left join t_dept on t_emp.eid = t_dept.did where t_emp.eid = #{eid}
</select>
```

### 方法3：分步查询（推荐）

#### 1. 查询员工信息

- select：设置分布查询的sql的唯一标识（namespace.SQLId或mapper接口的全类名.方法名）
- column：设置分步查询的条件

```java
//EmpMapper里的方法
/**
 * 通过分步查询，员工及所对应的部门信息
 * 分步查询第一步：查询员工信息
 * @param  
 * @return com.atguigu.mybatis.pojo.Emp
 * @date 2022/2/27 20:17
 */
Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);
```

```xml
<resultMap id="empAndDeptByStepResultMap" type="Emp">
	<id property="eid" column="eid"></id>
	<result property="empName" column="emp_name"></result>
	<result property="age" column="age"></result>
	<result property="sex" column="sex"></result>
	<result property="email" column="email"></result>
	<association property="dept"
				 select="com.atguigu.mybatis.mapper.DeptMapper.getEmpAndDeptByStepTwo"
				 column="did"></association>
</resultMap>
<!--Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);-->
<select id="getEmpAndDeptByStepOne" resultMap="empAndDeptByStepResultMap">
	select * from t_emp where eid = #{eid}
</select>
```

#### 2. 查询部门信息

```java
//DeptMapper里的方法
/**
 * 通过分步查询，员工及所对应的部门信息
 * 分步查询第二步：通过did查询员工对应的部门信息
 * @param
 * @return com.atguigu.mybatis.pojo.Emp
 * @date 2022/2/27 20:23
 */
Dept getEmpAndDeptByStepTwo(@Param("did") Integer did);
```

```xml
<!--此处的resultMap仅是处理字段和属性的映射关系-->
<resultMap id="EmpAndDeptByStepTwoResultMap" type="Dept">
	<id property="did" column="did"></id>
	<result property="deptName" column="dept_name"></result>
</resultMap>
<!--Dept getEmpAndDeptByStepTwo(@Param("did") Integer did);-->
<select id="getEmpAndDeptByStepTwo" resultMap="EmpAndDeptByStepTwoResultMap">
	select * from t_dept where did = #{did}
</select>
```

这样就可以通过select 将查询的结果赋值给第一个的property dept 对象

一共执行两个sql

![image-20220310165544419](https://image.imxyu.cn/file/image-20220310165544419.png)



## 延迟加载（分步查询的优点）

使用分步查询虽然写了两个sql，但是这两个sql可以作为单独的sql去调用执行，而且分步查询可以通过懒加载的配置进行按需执行，如下：

- 分步查询的优点：可以实现延迟加载，但是必须在核心配置文件中设置全局配置信息：
 - lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载  （默认false）
   - aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性。 否则，每个属性会按需加载  （默认false）
- 此时就可以实现按需加载，获取的数据是什么，就只会执行相应的sql。此时可通过association和collection中的fetchType属性设置当前的分步查询是否使用延迟加载，fetchType="lazy(延迟加载)|eager(立即加载)"

> 当我们需要延迟加载的话，需要将aggressiveLazyLoading设置为false，将lazyLoadingEnabled 设置为true 即可
>
> 而这两个都默认为false， 因此我们只需要将lazyLoadingEnabled  设置为true即可

```xml
<settings>
	<!--开启延迟加载-->
	<setting name="lazyLoadingEnabled" value="true"/>
</settings>
```

```java
@Test
public void getEmpAndDeptByStepOne() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	Emp emp = mapper.getEmpAndDeptByStepOne(1);
	System.out.println(emp.getEmpName());
}
```

- 关闭延迟加载，两条SQL语句都运行了![](https://image.imxyu.cn/file/%E5%BB%B6%E8%BF%9F%E5%8A%A0%E8%BD%BD%E6%B5%8B%E8%AF%951.png)
- 开启延迟加载，只运行获取emp的SQL语句
  ![](https://image.imxyu.cn/file/%E5%BB%B6%E8%BF%9F%E5%8A%A0%E8%BD%BD%E6%B5%8B%E8%AF%952.png)

```java
@Test
public void getEmpAndDeptByStepOne() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	Emp emp = mapper.getEmpAndDeptByStepOne(1);
	System.out.println(emp.getEmpName());
	System.out.println("----------------");
	System.out.println(emp.getDept());
}
```

- 开启后，需要用到查询dept的时候才会调用相应的SQL语句![](https://image.imxyu.cn/file/%E5%BB%B6%E8%BF%9F%E5%8A%A0%E8%BD%BD%E6%B5%8B%E8%AF%953.png)

上面的lazyLoadingEnabled  是全局属性，当设置完true 之后所有的都会延迟加载，如果我们有的sql语句不想延迟加载的话，就可以设置fetchType 属性单独指定

，当然如果我们lazyLoadingEnabled  不设置的话默认为false, 因为这是全局属性，所有的sql都会立即执行，这个fetchType类型就没有用。这里需要想明白

- fetchType：当开启了全局的延迟加载之后，可以通过该属性手动控制延迟加载的效果，fetchType="lazy(延迟加载)|eager(立即加载)"

```xml
<resultMap id="empAndDeptByStepResultMap" type="Emp">
	<id property="eid" column="eid"></id>
	<result property="empName" column="emp_name"></result>
	<result property="age" column="age"></result>
	<result property="sex" column="sex"></result>
	<result property="email" column="email"></result>
	<association property="dept"
				 select="com.atguigu.mybatis.mapper.DeptMapper.getEmpAndDeptByStepTwo"
				 column="did"
				 fetchType="lazy"></association>
</resultMap>

```



## 一对多映射处理

```java
public class Dept {
    private Integer did;
    private String deptName;
    private List<Emp> emps; //在多的一方设置list集合
	//...构造器、get、set方法等
}
```

### collection

- collection：用来处理一对多的映射关系
- ofType：表示该属性对应的集合中存储的数据的类型

```xml
<resultMap id="DeptAndEmpResultMap" type="Dept">
	<id property="did" column="did"></id>
	<result property="deptName" column="dept_name"></result>
	<collection property="emps" ofType="Emp">
		<id property="eid" column="eid"></id>
		<result property="empName" column="emp_name"></result>
		<result property="age" column="age"></result>
		<result property="sex" column="sex"></result>
		<result property="email" column="email"></result>
	</collection>
</resultMap>
<!--Dept getDeptAndEmp(@Param("did") Integer did);-->
<select id="getDeptAndEmp" resultMap="DeptAndEmpResultMap">
	select * from t_dept left join t_emp on t_dept.did = t_emp.did where t_dept.did = #{did}
</select>
```

### 分步查询（推荐）

####  1. 查询部门信息

```java
/**
 * 通过分步查询，查询部门及对应的所有员工信息
 * 分步查询第一步：查询部门信息
 * @param did 
 * @return com.atguigu.mybatis.pojo.Dept
 * @date 2022/2/27 22:04
 */
Dept getDeptAndEmpByStepOne(@Param("did") Integer did);
```

```xml
<resultMap id="DeptAndEmpByStepOneResultMap" type="Dept">
	<id property="did" column="did"></id>
	<result property="deptName" column="dept_name"></result>
	<collection property="emps"
				select="com.atguigu.mybatis.mapper.EmpMapper.getDeptAndEmpByStepTwo"
				column="did"></collection>
</resultMap>
<!--Dept getDeptAndEmpByStepOne(@Param("did") Integer did);-->
<select id="getDeptAndEmpByStepOne" resultMap="DeptAndEmpByStepOneResultMap">
	select * from t_dept where did = #{did}
</select>
```

#### 2. 根据部门id查询部门中的所有员工

```java
/**
 * 通过分步查询，查询部门及对应的所有员工信息
 * 分步查询第二步：根据部门id查询部门中的所有员工
 * @param did
 * @return java.util.List<com.atguigu.mybatis.pojo.Emp>
 * @date 2022/2/27 22:10
 */
List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);
```

```xml
<!--List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);-->
<select id="getDeptAndEmpByStepTwo" resultType="Emp">
	select * from t_emp where did = #{did}
</select>
```

# 动态SQL

- Mybatis框架的动态SQL技术是一种根据特定条件动态拼装SQL语句的功能，它存在的意义是为了解决拼接SQL语句字符串时的痛点问题

## if

- if标签可通过test属性（即传递过来的数据）的表达式进行判断，若表达式的结果为true，则标签中的内容会执行；反之标签中的内容不会执行
- 在where后面添加一个恒成立条件`1=1`
 - 这个恒成立条件并不会影响查询的结果
   - 这个`1=1`可以用来拼接`and`语句，例如：当empName为null时
    - 如果不加上恒成立条件，则SQL语句为`select * from t_emp where and age = ? and sex = ? and email = ?`，此时`where`会与`and`连用，SQL语句会报错
      - 如果加上一个恒成立条件，则SQL语句为`select * from t_emp where 1= 1 and age = ? and sex = ? and email = ?`，此时不报错

```xml
<!--List<Emp> getEmpByCondition(Emp emp);-->
<select id="getEmpByCondition" resultType="Emp">
	select * from t_emp where 1=1
	<if test="empName != null and empName !=''">
		and emp_name = #{empName}
	</if>
	<if test="age != null and age !=''">
		and age = #{age}
	</if>
	<if test="sex != null and sex !=''">
		and sex = #{sex}
	</if>
	<if test="email != null and email !=''">
		and email = #{email}
	</if>
</select>
```

## where

- where和if一般结合使用：
 - 若where标签中的if条件都不满足，则where标签没有任何功能，即不会添加where关键字  
   - 若where标签中的if条件满足，则where标签会自动添加where关键字，并将条件**最前方**多余的and/or去掉  

```xml
<!--List<Emp> getEmpByCondition(Emp emp);-->
<select id="getEmpByCondition" resultType="Emp">
	select * from t_emp
	<where>
		<if test="empName != null and empName !=''">
			emp_name = #{empName}
		</if>
		<if test="age != null and age !=''">
			and age = #{age}
		</if>
		<if test="sex != null and sex !=''">
			and sex = #{sex}
		</if>
		<if test="email != null and email !=''">
			and email = #{email}
		</if>
	</where>
</select>
```

- 注意：where标签不能去掉条件后多余的and/or

	```xml
	<!--这种用法是错误的，只能去掉条件前面的and/or，条件后面的不行-->
	<if test="empName != null and empName !=''">
	emp_name = #{empName} and
	</if>
	<if test="age != null and age !=''">
		age = #{age}
	</if>
	```

## trim

刚刚说了where 标签只能将我们前面的and或者or去掉，如果我们想在后面加or 或者and就要用到这个标签

- trim用于去掉或添加标签中的内容  
- 常用属性
 - prefix：在trim标签中的内容的前面添加某些内容  
   - suffix：在trim标签中的内容的后面添加某些内容 
   - prefixOverrides：在trim标签中的内容的前面去掉某些内容  
   - suffixOverrides：在trim标签中的内容的后面去掉某些内容
- 若trim中的标签都不满足条件，则trim标签没有任何效果，也就是只剩下`select * from t_emp`

```xml
<!--List<Emp> getEmpByCondition(Emp emp);-->
<select id="getEmpByCondition" resultType="Emp">
	select * from t_emp
	<trim prefix="where" suffixOverrides="and|or">
		<if test="empName != null and empName !=''">
			emp_name = #{empName} and
		</if>
		<if test="age != null and age !=''">
			age = #{age} and
		</if>
		<if test="sex != null and sex !=''">
			sex = #{sex} or
		</if>
		<if test="email != null and email !=''">
			email = #{email}
		</if>
	</trim>
</select>
```

```java
//测试类
@Test
public void getEmpByCondition() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	DynamicSQLMapper mapper = sqlSession.getMapper(DynamicSQLMapper.class);
	List<Emp> emps= mapper.getEmpByCondition(new Emp(null, "张三", null, null, null, null));
	System.out.println(emps);
}
```

![](https://image.imxyu.cn/file/trim%E6%B5%8B%E8%AF%95%E7%BB%93%E6%9E%9C.png)

## choose、when、otherwise

- `choose、when、otherwise`相当于`if...else if..else`
- when至少要有一个，otherwise至多只有一个

```xml
<select id="getEmpByChoose" resultType="Emp">
	select * from t_emp
	<where>
		<choose>
			<when test="empName != null and empName != ''">
				emp_name = #{empName}
			</when>
			<when test="age != null and age != ''">
				age = #{age}
			</when>
			<when test="sex != null and sex != ''">
				sex = #{sex}
			</when>
			<when test="email != null and email != ''">
				email = #{email}
			</when>
			<otherwise>
				did = 1
			</otherwise>
		</choose>
	</where>
</select>
```

```java
@Test
public void getEmpByChoose() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	DynamicSQLMapper mapper = sqlSession.getMapper(DynamicSQLMapper.class);
	List<Emp> emps = mapper.getEmpByChoose(new Emp(null, "张三", 23, "男", "123@qq.com", null));
	System.out.println(emps);
}
```

![](https://image.imxyu.cn/file/choose%E6%B5%8B%E8%AF%95%E7%BB%93%E6%9E%9C.png)

- 相当于`if a else if b else if c else d`，只会执行其中一个

## foreach

- 属性：  
 - collection：设置要循环的数组或集合  
   - item：表示集合或数组中的每一个数据  
   - separator：设置循环体之间的分隔符，分隔符前后默认有一个空格，如` , `
   - open：设置foreach标签中的内容的开始符  
   - close：设置foreach标签中的内容的结束符
- 批量删除

```xml

<!--int deleteMoreByArray(Integer[] eids);-->
<delete id="deleteMoreByArray">
	delete from t_emp where eid in
	<foreach collection="eids" item="eid" separator="," open="(" close=")">
		#{eid}
	</foreach>
</delete>

```

```java
@Test
public void deleteMoreByArray() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	DynamicSQLMapper mapper = sqlSession.getMapper(DynamicSQLMapper.class);
	int result = mapper.deleteMoreByArray(new Integer[]{6, 7, 8, 9});
	System.out.println(result);
}
```
![](https://image.imxyu.cn/file/foreach%E6%B5%8B%E8%AF%95%E7%BB%93%E6%9E%9C1.png)

- 批量添加

```xml

<!--int insertMoreByList(@Param("emps") List<Emp> emps);-->
<insert id="insertMoreByList">
	insert into t_emp values
	<foreach collection="emps" item="emp" separator=",">
		(null,#{emp.empName},#{emp.age},#{emp.sex},#{emp.email},null)
	</foreach>
</insert>


```

```java
@Test
public void insertMoreByList() {
	SqlSession sqlSession = SqlSessionUtils.getSqlSession();
	DynamicSQLMapper mapper = sqlSession.getMapper(DynamicSQLMapper.class);
	Emp emp1 = new Emp(null,"a",1,"男","123@321.com",null);
	Emp emp2 = new Emp(null,"b",1,"男","123@321.com",null);
	Emp emp3 = new Emp(null,"c",1,"男","123@321.com",null);
	List<Emp> emps = Arrays.asList(emp1, emp2, emp3);
	int result = mapper.insertMoreByList(emps);
	System.out.println(result);
}
```
![](https://image.imxyu.cn/file/foreach%E6%B5%8B%E8%AF%95%E7%BB%93%E6%9E%9C2.png)

## SQL片段

- sql片段，可以记录一段公共sql片段，在使用的地方通过include标签进行引入
- 声明sql片段：`<sql>`标签

```xml
<sql id="empColumns">eid,emp_name,age,sex,email</sql>
```

- 引用sql片段：`<include>`标签

```xml
<!--List<Emp> getEmpByCondition(Emp emp);-->
<select id="getEmpByCondition" resultType="Emp">
	select <include refid="empColumns"></include> from t_emp
</select>
```

# MyBatis的缓存

缓存是只针对查询功能的，如果缓存中有则不会查询数据库

## MyBatis的一级缓存

一级缓存是默认开启的

- 一级缓存是SqlSession级别的，**通过同一个SqlSession查询的数据会被缓存**，下次查询相同的数据，就会从缓存中直接获取，不会从数据库重新访问  
- 使一级缓存失效的四种情况：  

	1. 不同的SqlSession对应不同的一级缓存  
	
	2. 同一个SqlSession但是查询条件不同
	
	3. 同一个SqlSession两次查询期间执行了任何一次增删改操作
	
	4. 同一个SqlSession两次查询期间手动清空了缓存
	
	   sqlSession.clearCache();

```java
    @Test
    public void testCache(){
        SqlSession sqlSession = SqlSessionUtils.getSqlSession();
        CacheMapper mapper = sqlSession.getMapper(CacheMapper.class);
        Emp emp1 = mapper.getEmpById(3);
        System.out.println(emp1);
        System.out.println("========第二次调用========从缓存中取数据");
        Emp emp2 = mapper.getEmpById(3);
        System.out.println(emp2);

        System.out.println("\n========即使用的不是同一个Mapper，也同样从缓存中取(同一个sqlsession)========");
        CacheMapper mapper2 = sqlSession.getMapper(CacheMapper.class);
        Emp empByMapper2 = mapper2.getEmpById(3);
        System.out.println(empByMapper2);

        System.out.println("\n========一级缓存的范围在sqlsession中，换一个新的sqlsession就会再次用sql读取数据========");
        SqlSession sqlSession2 = SqlSessionUtils.getSqlSession();
        CacheMapper mapper2BySqlSession2 = sqlSession2.getMapper(CacheMapper.class);
        System.out.println(mapper2BySqlSession2.getEmpById(3));
    }

```

![在这里插入图片描述](https://image.imxyu.cn/file/df15c883f66244cbae9ea4b5343674b7.png)

失效的4种情况就不演示了



## MyBatis的二级缓存

二级缓存是默认关闭的，需要手动开启 

- 二级缓存是SqlSessionFactory级别，**通过同一个SqlSessionFactory**创建的SqlSession查询的结果会被缓存；此后若再次执行相同的查询语句，结果就会从缓存中获取  
- 二级缓存开启的条件

	1. 在核心配置文件中，设置全局配置属性cacheEnabled="true"，默认为true，不需要设置
	2. 在xml映射文件中设置标签<cache />
	3. 二级缓存必须在SqlSession关闭close或提交commit之后有效，没提交之前都是保存在一级缓存当中
	4. 查询的数据所转换的实体类类型必须实现序列化的接口
- 使二级缓存失效的情况：两次查询之间执行了任意的增删改，**会使一级和二级缓存同时失效**

测试：（首先我们没有close）

```java
    @Test
    public void testCacheTwo(){
        //这里不能用工具类了，因为每次都会创建新的sqlsessionfactory
//        SqlSession sqlSession = SqlSessionUtils.getSqlSession();
//        CacheMapper mapper = sqlSession.getMapper(CacheMapper.class);

        //只要是同一个sqlsessionfactory获得的sqlsession就可以
        try {
            InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
            SqlSession sqlSession1 = sqlSessionFactory.openSession(true);
            CacheMapper mapper1 = sqlSession1.getMapper(CacheMapper.class);
            System.out.println(mapper1.getEmpById(1));

            System.out.println("Cache Hit Ratio：缓存命中率，指的是在缓存中有没有这条数据");
            System.out.println("=====二级缓存未打开，没从缓存中获取数据=====");
            SqlSession sqlSession2 = sqlSessionFactory.openSession(true);
            CacheMapper mapper2 = sqlSession2.getMapper(CacheMapper.class);
            System.out.println(mapper2.getEmpById(1));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

```

测试结果：

![在这里插入图片描述](https://image.imxyu.cn/file/354299e254d744d99542fb0a87054a8b.png)

我们把sqlsession关闭掉，再来测试

![在这里插入图片描述](https://image.imxyu.cn/file/1753773af6a748a58169953bca8b16f3.png)

## 二级缓存的相关配置

- 在mapper配置文件中添加的cache标签可以设置一些属性

- eviction属性：缓存回收策略  （因为缓存是放在内存中的，不可能一直往里存）
	- LRU（Least Recently Used） – 最近最少使用的：移除最长时间不被使用的对象。  
	- FIFO（First in First out） – 先进先出：按对象进入缓存的顺序来移除它们。  
	- SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。  
	- WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
	- 默认的是 LRU
- flushInterval属性：刷新间隔，单位毫秒
	- 默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句（增删改）时刷新
- size属性：引用数目，正整数
	- 代表缓存最多可以存储多少个对象，太大容易导致内存溢出
- readOnly属性：只读，true/false
	- true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。  如果修改了下次别人从缓存中获取就和数据库中的不一样了就变成你修改后的了，就会出现问题
	- false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是false
## MyBatis缓存查询的顺序

先从大范围搜索，再去小的范围搜索

- 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用  
- 如果二级缓存没有命中，再查询一级缓存  
- 如果一级缓存也没有命中，则查询数据库  
- SqlSession关闭之后，一级缓存中的数据会写入二级缓存

> 因为二级缓存是sqlSessionFactory 级别的，范围是比sqlSession大，所有的sqlSession关闭之后都会放二级缓存中，因此查询的时候先去二级缓存中查，如果没有的话再去1级查

## 整合第三方缓存EHCache（了解） 

当然mybatis毕竟不是做缓存的，所以缓存这方面可能不太专业，所以我们可以整合第三方缓存

注意第三方缓存只能用来替代**二级缓存**的

### 添加依赖
```xml
<!-- Mybatis EHCache整合包 -->
<dependency>
	<groupId>org.mybatis.caches</groupId>
	<artifactId>mybatis-ehcache</artifactId>
	<version>1.2.1</version>
</dependency>
<!-- slf4j日志门面的一个具体实现 -->
<dependency>
	<groupId>ch.qos.logback</groupId>
	<artifactId>logback-classic</artifactId>
	<version>1.2.3</version>
</dependency>
```
> 日志门面的意思就是提供一个日志接口，slf4j-api 就是提供了一个日志的接口，而logback-classic则是sl4fj 的具体实现
>
> 因为maven当我们加入ehcache-mybatis整合包时，肯定是需要依赖ehcache 的核心包的，所以会自动把ehcache的核心包加入进来。同理logback-classic需要slf4j-api 这个接口，因此也会被导入进来

### 各个jar包的功能

| jar包名称 | 作用 |
| --- | --- |
| mybatis-ehcache | Mybatis和EHCache的整合包 |
| ehcache | EHCache核心包 |
| slf4j-api | SLF4J日志门面包 |
| logback-classic | 支持SLF4J门面接口的一个具体实现 |

### 创建EHCache的配置文件ehcache.xml
- 名字必须叫`ehcache.xml`
```xml
<?xml version="1.0" encoding="utf-8" ?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
    <!-- 缓存在磁盘中保存路径 -->
    <diskStore path="D:\atguigu\ehcache"/>
    <defaultCache
            maxElementsInMemory="1000"
            maxElementsOnDisk="10000000"
            eternal="false"
            overflowToDisk="true"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
    </defaultCache>
</ehcache>
```
### 设置二级缓存的类型
- 在xxxMapper.xml文件中设置二级缓存类型
```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```
### 加入logback日志
- 存在**SLF4J**时，作为简易日志的log4j将失效，此时我们需要借助SLF4J的具体实现logback来打印日志。创建logback的配置文件`logback.xml`，名字固定，不可改变
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">
    <!-- 指定日志输出的位置 -->
    <appender name="STDOUT"
              class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- 日志输出的格式 -->
            <!-- 按照顺序分别是：时间、日志级别、线程名称、打印日志的类、日志主体内容、换行 -->
            <pattern>[%d{HH:mm:ss.SSS}] [%-5level] [%thread] [%logger] [%msg]%n</pattern>
        </encoder>
    </appender>
    <!-- 设置全局日志级别。日志级别按顺序分别是：DEBUG、INFO、WARN、ERROR -->
    <!-- 指定任何一个日志级别都只打印当前级别和后面级别的日志。 -->
    <root level="DEBUG">
        <!-- 指定打印日志的appender，这里通过“STDOUT”引用了前面配置的appender -->
        <appender-ref ref="STDOUT" />
    </root>
    <!-- 根据特殊需求指定局部日志级别 -->
    <logger name="com.atguigu.crowd.mapper" level="DEBUG"/>
</configuration>
```
### EHCache配置文件说明
| 属性名 | 是否必须 | 作用 |
| --- | --- | --- |
| maxElementsInMemory | 是 | 在内存中缓存的element的最大数目 |
| maxElementsOnDisk | 是 | 在磁盘上缓存的element的最大数目，若是0表示无穷大 |
| eternal | 是 | 设定缓存的elements是否永远不过期。 如果为true，则缓存的数据始终有效， 如果为false那么还要根据timeToIdleSeconds、timeToLiveSeconds判断 |
| overflowToDisk | 是 | 设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上 |
| timeToIdleSeconds | 否 | 当缓存在EhCache中的数据前后两次访问的时间超过timeToIdleSeconds的属性取值时， 这些数据便会删除，默认值是0,也就是可闲置时间无穷大 |
| timeToLiveSeconds | 否 | 缓存element的有效生命期，默认是0.,也就是element存活时间无穷大 |
| diskSpoolBufferSizeMB | 否 | DiskStore(磁盘缓存)的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区 |
| diskPersistent | 否 | 在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false |
| diskExpiryThreadIntervalSeconds | 否 | 磁盘缓存的清理线程运行间隔，默认是120秒。每个120s， 相应的线程会进行一次EhCache中数据的清理工作 |
| memoryStoreEvictionPolicy | 否 | 当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。 默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出 |
# MyBatis的逆向工程 
- 正向工程：先创建Java实体类，由框架负责根据实体类生成数据库表。Hibernate是支持正向工程的
- 逆向工程：先创建数据库表，由框架负责根据数据库表，反向生成如下资源：  
	- Java实体类  
	- Mapper接口  
	- Mapper映射文件
## 创建逆向工程的步骤
### 1）添加依赖和插件
```xml
<dependencies>
	<!-- MyBatis核心依赖包 -->
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.5.9</version>
	</dependency>
	<!-- junit测试 -->
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.13.2</version>
		<scope>test</scope>
	</dependency>
	<!-- MySQL驱动 -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>8.0.27</version>
	</dependency>
	<!-- log4j日志 -->
	<dependency>
		<groupId>log4j</groupId>
		<artifactId>log4j</artifactId>
		<version>1.2.17</version>
	</dependency>
</dependencies>
<!-- 控制Maven在构建过程中相关配置 -->
<build>
	<!-- 构建过程中用到的插件 -->
	<plugins>
		<!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
		<plugin>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-maven-plugin</artifactId>
			<version>1.3.0</version>
			<!-- 插件的依赖 -->
			<dependencies>
				<!-- 逆向工程的核心依赖 -->
				<dependency>
					<groupId>org.mybatis.generator</groupId>
					<artifactId>mybatis-generator-core</artifactId>
					<version>1.3.2</version>
				</dependency>
				<!-- 数据库连接池 -->
				<dependency>
					<groupId>com.mchange</groupId>
					<artifactId>c3p0</artifactId>
					<version>0.9.2</version>
				</dependency>
				<!-- MySQL驱动 -->
				<dependency>
					<groupId>mysql</groupId>
					<artifactId>mysql-connector-java</artifactId>
					<version>8.0.27</version>
				</dependency>
			</dependencies>
		</plugin>
	</plugins>
</build>
```
### 2）创建MyBatis的核心配置文件

在[src](https://so.csdn.net/so/search?q=src&spm=1001.2101.3001.7020)/main/resources下创建mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties"/>
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
创建[jdbc](https://so.csdn.net/so/search?q=jdbc&spm=1001.2101.3001.7020).properties文件

```java
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8
jdbc.username=root
jdbc.password=写你的数据库密码

```

创建[log4j](https://so.csdn.net/so/search?q=log4j&spm=1001.2101.3001.7020).xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                     xsi:schemaLocation="http://jakarta.apache.org/log4j/ ">
    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
        <param name="Encoding" value="UTF-8"/>
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS}
%m  (%F:%L) \n"/>
        </layout>
    </appender>
    <logger name="java.sql">
        <level value="debug"/>
    </logger>
    <logger name="org.apache.ibatis">
        <level value="info"/>
    </logger>
    <root>
        <level value="debug"/>
        <appender-ref ref="STDOUT"/>
    </root>
</log4j:configuration>

```



### 3）创建逆向工程的配置文件

- 文件名必须是：`generatorConfig.xml`
- ⭕️ `"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd"`这条报红不用管。
  ⭕️ 注意：mac环境下，\要修改为/，windows才是targetProject="./src/main/java
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--
    targetRuntime: 执行生成的逆向工程的版本
    MyBatis3Simple: 生成基本的CRUD（清新简洁版）
    MyBatis3: 生成带条件的CRUD（奢华尊享版）
    -->
    <context id="DB2Tables" targetRuntime="MyBatis3Simple">
        <!-- 数据库的连接信息 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- javaBean的生成策略-->
        <javaModelGenerator targetPackage="com.atguigu.mybatis.pojo" targetProject=".\src\main\java">
             <!--这个是是否允许子包，就是上面的targetPackage 中的. true的话是对应的很多子包，false则只是一个目录的名字-->
            <property name="enableSubPackages" value="true" /> 
            <!-- 数据库中如果列字段包含空格的话，去除掉空格再转换成bean-->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- SQL映射文件的生成策略 -->
        <sqlMapGenerator targetPackage="com.atguigu.mybatis.mapper"
                         targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- Mapper接口的生成策略 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.atguigu.mybatis.mapper" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- 逆向分析的表 -->
        <!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
        <!-- domainObjectName属性指定生成出来的实体类的类名 -->
        <table tableName="t_emp" domainObjectName="Emp"/>
        <table tableName="t_dept" domainObjectName="Dept"/>
    </context>
</generatorConfiguration>
```
> 如果我们使用的是dtd 约束头的话，<!DOCTYPE 后面的这个generatorConfiguration 就是代表的根标签

## 清新简洁版

### 执行MBG插件的generate目标

- ![](https://image.imxyu.cn/file/%E6%89%A7%E8%A1%8CMBG%E6%8F%92%E4%BB%B6%E7%9A%84generate%E7%9B%AE%E6%A0%87.png)
- 如果出现报错：`Exception getting JDBC Driver`，可能是pom.xml中，数据库驱动配置错误
	- dependency中的驱动![](https://image.imxyu.cn/file/dependency%E4%B8%AD%E7%9A%84%E9%A9%B1%E5%8A%A8.png)
	- mybatis-generator-maven-plugin插件中的驱动![](https://image.imxyu.cn/file/%E6%8F%92%E4%BB%B6%E4%B8%AD%E7%9A%84%E9%A9%B1%E5%8A%A8.png)
	- 两者的驱动版本应该相同

操作前的文件目录：

![在这里插入图片描述](https://image.imxyu.cn/file/1e882730247a4c2d9f694c8cb994b03b.png)

- 执行结果

  ![在这里插入图片描述](https://image.imxyu.cn/file/52094e143daf4e839cfd17068065e8eb.png)

[pojo](https://so.csdn.net/so/search?q=pojo&spm=1001.2101.3001.7020)中自动生成属性和get/set方法



![在这里插入图片描述](https://image.imxyu.cn/file/3dd0505ee66347fcbcda6c5eb57d337d.png)

同样的mapper 也提供了基础的增删改查方法

## 奢华版（常用）

更改[参数](https://so.csdn.net/so/search?q=参数&spm=1001.2101.3001.7020)为MyBatis3: 生成带条件的CRUD

![在这里插入图片描述](https://image.imxyu.cn/file/3cb4df8386464f2da024fdcebf5ca6ef.png)

自动生成的Mapper接口中的方法

```java
public interface EmpMapper {
	// 根据条件计数
    int countByExample(EmpExample example);

	//根据条件删除
    int deleteByExample(EmpExample example);
	//根据主键删除
    int deleteByPrimaryKey(Integer eid);

	//普通插入
    int insert(Emp record);
	//选择性插入：没写的就是null
    int insertSelective(Emp record);

	//根据条件查询
    List<Emp> selectByExample(EmpExample example);
	//根据主键查询
    Emp selectByPrimaryKey(Integer eid);

	//根据条件选择性修改：
    int updateByExampleSelective(@Param("record") Emp record, @Param("example") EmpExample example);
    //根据条件修改
    int updateByExample(@Param("record") Emp record, @Param("example") EmpExample example);
	//根据主键选择性修改
    int updateByPrimaryKeySelective(Emp record);
	//根据主键修改
    int updateByPrimaryKey(Emp record);
}

```

### QBC风格测试

QBC风格就是通过拼接条件进行查询和修改，通过生成的example 可以进行条件拼接 对应的方法就是上面mapper 中的有关example的条件查询和修改方法

给Emp和Dept 的pojo重写toString()，再加一个空参构造器、一个有参构造器，然后就可以开始测试了。

```java
    @Test
    public void test(){
        try {
            InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
            SqlSession sqlSession = sqlSessionFactory.openSession();
            EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);

            //查询所有数据
            System.out.println("\n--------->查询所有数据");
            List<Emp> emps = mapper.selectByExample(null); // 没有查询条件就相当于查询所有数据
            emps.forEach(emp -> System.out.println(emp));

            //根据条件查询 QBC: Query by Criteria
            EmpExample example = new EmpExample();

            //名字叫Bela的
            System.out.println("\n--------->根据条件查询");
            example.createCriteria().andEmpNameEqualTo("Bela");
            List<Emp> emps1 = mapper.selectByExample(example);
            emps1.forEach(emp -> System.out.println(emp));

            //链式添加条件
            System.out.println("\n--------->链式添加条件");
            example.createCriteria().andEmpNameEqualTo("Bela").andAgeEqualTo(33);
            List<Emp> emps2 = mapper.selectByExample(example);
            emps2.forEach(emp -> System.out.println(emp));

            //两个条件用or连接
            System.out.println("\n--------->两个条件用or连接");
            example.createCriteria().andAgeLessThan(30);
            example.or().andDidIsNotNull();
            List<Emp> emps3 = mapper.selectByExample(example);
            emps3.forEach(emp -> System.out.println(emp));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

```

#### 查询所有数据

![在这里插入图片描述](https://image.imxyu.cn/file/a187ec53ccab4561afe247c1c8bf55f6.png)

#### 根据条件查询

根据生成的Example ，可以进行拼接条件进行查询

![在这里插入图片描述](https://image.imxyu.cn/file/e668505c345a43abbb226fdf49a1fb34.png)

#### 链式添加条件

![在这里插入图片描述](https://image.imxyu.cn/file/8339aec73c1f4edf9258acfccbdd02af.png)



#### 两个条件用or连接

![在这里插入图片描述](https://image.imxyu.cn/file/66ffd0eeab7c4109a4b607d67a976be3.png)

#### 测试方法自动生成的Mapper接口中的方法：修改

主要注意一下带有selective 的方法对于空的值不会作为查询条件或者修改的条件，里面会经过if null 的判断的

```java
    @Test
    public void test2(){
        try {
            InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
            SqlSession sqlSession = sqlSessionFactory.openSession();
            EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);

            // 原始id=2的数据
            System.out.println("原始id=1的数据"+mapper.selectByPrimaryKey(2));

            // 根据主键修改
            mapper.updateByPrimaryKey(new Emp(2,"改1",55,"男","6789@gamil.com",null));
            System.out.println("\n-----> 根据主键修改----->" + mapper.selectByPrimaryKey(1));

            // 根据主键选择性修改
            mapper.updateByPrimaryKeySelective(new Emp(2,"改2",55,null,"6789@gamil.com",null));
            System.out.println("\n-----> 根据主键选择性修改----->" + mapper.selectByPrimaryKey(1));

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

```

![在这里插入图片描述](https://image.imxyu.cn/file/c233f850227e4d8c8a831e4dc6104a60.png)

# 分页插件
## 分页插件使用步骤
### 添加依赖
```xml
<!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>5.2.0</version>
</dependency>
```
### 配置分页插件
- 在MyBatis的核心配置文件（mybatis-config.xml）中配置插件
- ![](https://image.imxyu.cn/file/%E9%85%8D%E7%BD%AE%E5%88%86%E9%A1%B5%E6%8F%92%E4%BB%B6.png)
```xml
<plugins>
	<!--设置分页插件-->
	<plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
</plugins>
```
## 分页插件的使用
### 开启分页功能
- 在查询功能之前使用`PageHelper.startPage(int pageNum, int pageSize)`开启分页功能
	- pageNum：当前页的页码  
	- pageSize：每页显示的条数
```java
@Test
public void testPageHelper() throws IOException {
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession(true);
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	//访问第一页，每页四条数据
	PageHelper.startPage(1,4);
	List<Emp> emps = mapper.selectByExample(null);
	emps.forEach(System.out::println);
}
```

![](https://image.imxyu.cn/file/%E5%88%86%E9%A1%B5%E6%B5%8B%E8%AF%95%E7%BB%93%E6%9E%9C.png)
### 分页相关数据
#### 方法一：直接输出
```java
@Test
public void testPageHelper() throws IOException {
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession(true);
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	//访问第一页，每页四条数据
	Page<Object> page = PageHelper.startPage(1, 4);
	List<Emp> emps = mapper.selectByExample(null);
	//在查询到List集合后，打印分页数据
	System.out.println(page);
}
```
- 分页相关数据：

	```
	Page{count=true, pageNum=1, pageSize=4, startRow=0, endRow=4, total=8, pages=2, reasonable=false, pageSizeZero=false}[Emp{eid=1, empName='admin', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=2, empName='admin2', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=3, empName='王五', age=12, sex='女', email='123@qq.com', did=3}, Emp{eid=4, empName='赵六', age=32, sex='男', email='123@qq.com', did=1}]
	```
#### 方法二使用PageInfo
- 在查询获取list集合之后，使用`PageInfo<T> pageInfo = new PageInfo<>(List<T> list, intnavigatePages)`获取分页相关数据
	- list：分页之后的数据  
	- navigatePages：导航分页的页码数
```java
    /**
     * limit    index，pagesize
     * index    当前页的起始索引
     * pageSize 每页显示的条数
     * pageNum  当前页的页码
     * 当前页的起始索引 = 每页条数 * 页码 - 1
     * index = pageNum * pageSize - 1
     *
     * 通过索引获得数据
     *
     * 使用MyBatis的分页插件，实现分页功能：
     * 1。需要在查询功能之前开启分页
     * PageHelper.startPage(2, 4);
     * 
     * 2。在查询功能之后获取分页相关信息
     *   PageInfo<Emp> pages = new PageInfo<>(emps, 5); 5表示导航分页的数量
     */
    @Test
    public void test2(){
        try {
            InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
            SqlSession sqlSession = sqlSessionFactory.openSession();
            EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);

            System.out.println("\n查询功能前开启分页");
            PageHelper.startPage(2, 4);
            List<Emp> emps = mapper.selectByExample(null);
            emps.forEach(emp -> System.out.println(emp));

            System.out.println("\n");
            PageInfo<Emp> pages = new PageInfo<>(emps, 5);
            System.out.println("PageInfo----->"+pages);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

```
分页相关数据：

```java
PageInfo{undefined
pageNum=8, pageSize=4, size=2, startRow=29, endRow=30, total=30, pages=8,
list=Page{count=true, pageNum=8, pageSize=4, startRow=28, endRow=32, total=30, pages=8, reasonable=false, pageSizeZero=false},
prePage=7, nextPage=0, isFirstPage=false, isLastPage=true, hasPreviousPage=true, hasNextPage=false, navigatePages=5, navigateFirstPage4, navigateLastPage8, navigatepageNums=[4, 5, 6, 7, 8]
}
常用数据:
pageNum:当前页的页码
pageSize:每页显示的条数
size:当前页显示的真实条数
total:总记录数
pages:总页数
prePage:上一页的页码
nextPage:下一页的页码
isFirstPage/isLastPage:是否为第一页/最后一页
hasPreviousPage/hasNextPage:是否存在上一页/下一页
navigatePages:导航分页的页码数
navigatepageNums:导航分页的页码，[1,2,3,4,5]

```

![在这里插入图片描述](https://image.imxyu.cn/file/57a62fc2ee454681ae1257732d359609.png)

![在这里插入图片描述](https://image.imxyu.cn/file/5f251ce79ad5411a93a34c7d74b0375d.png)