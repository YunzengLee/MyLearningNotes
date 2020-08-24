第一天  入门

- 入门
- 概述
- 环境搭建
- 入门案例
- 自定义

第二天  基本使用

- 单表crud操作
- 参数和返回值设置
- dao编写（DAO：data access object 数据访问层）
- 配置文件细节（几个标签的使用）

第三天  深入和多表

- 连接池
- 事务的控制及设计的方法
- 多表查询
  - 一对多
  - 多对多

第四天  mybatis的缓存和注解开发

- 加载时机（查询的时机）
- 一级 二级缓存
- 注解开发
  - 单表crud
  - 多表查询

### 第一天

#### 1 框架

- 软件开发中的一套解决方案，不同框架解决不同的问题。

- 封装了很多细节，使开发者可以使用极简的方式实现功能，提高开发效率。

#### 2 三层架构

- 表现层：展示数据
- 业务层：处理业务需求
- 持久层：和数据库交互

#### 3 持久层技术解决方案

- JDBC：
  - Connection
  - PreparedStatement
  - ResultSet
- Spring的JDBCTemplate
  - 对JDBC的简单封装
- Apache的Utils（也是对JDBC的简答封装）

以上都不是框架，JDBC是规范，后两个都只是工具类。

#### 4 概述

基于java 的持久层框架，封装了JDBC，使开发者只需关注sql本身，无需处理加载驱动，创建连接，创建statement等过程。

通过xml或注解的方式将要执行的statement配置起来，并通过java对象和statement中sql的动态参数进行映射，生成最终要执行的sql语句，最后由mybatis执行sql并将结果映射为java对象返回。

采用ORM思想解决了实体和数据库映射的问题，对JDBC进行了封装。

ORM就是把数据库表和实体类及其属性对应起来，通过操作实体类就实现操作数据库。

#### 5 入门

##### 使用xml

- Maven创建工程（不使用骨架）
- 配置pom.xml

```xml
<dependencies>
	<dependency>
    	<groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.5</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.19</version>
    </dependency>
    <dependency>
    	<groupId>log4j</groupId>  日志相关
        <artifactId>log4j</artifactId>
        <version></version>
    </dependency>
    <dependency>  单元测试
    	<groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version></version>
    </dependency>
</dependencies>
```

- 在com.xxx.domain文件夹下创建Bean实体类

  ```java
  public class User implements Serializable{
      //与数据库表的字段一致
      private Integer id;
      private String name;
      public void getId(){
          return id;
      }
      public void setId(Integer id){
          this.id=id;
      }
      public void getName(){
          return name;
      }
      public void setName(String name){
          this.name=name;
      }
      
  }
  ```

- 在com.xxx.dao文件夹下创建dao（也叫mapper）接口类。

  ```java
  public interface IUserDao{//用户的持久层接口
      List<User> findall(); //定义一个查询所有用户的方法
  }
  ```

- 在main.resources下创建配置文件SqlMyConfig.xml

  ```xml
  <!--头部信息-->
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <!--mybatis的主配置文件-->
  <configuration>
  	<!--配置环境-->
      <environments default="mysql">
      	<environment id="mysql">
          	<transactionManager type="JDBC"><!---配置事务类型-->
              </transactionManager>
              <!--配置数据源，也叫连接池 POOLED表示使用连接池-->
              <dataSource type="POOLED">
              	<!--基本信息-->
                  <property name="driver" value="com.mysql.jdbc.driver" />
                  <property name="url" value="jdbc:mysql://localhost:3306/databasename" />
                  <property name="username" value="root" />
                  <property name="password" value="1234" />
              </dataSource>
          </environment>
      </environments>
      
      <!--指定映射配置文件的位置，也就是每个DAO独立的配置文件-->
      <mappers>
      	<mapper resources="com/xxx/dao/IUserDao.xml" />
          <!--注意，此处的路径默认指的是在main/resources/文件夹下的com/xxx/dao/...-->
      </mappers>
  </configuration>
  ```

  

- 在main/resources文件夹下创建com/xxx/dao/IUserDao.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.xxx.dao.IUserDao">
  	<!--配置查询所有的sql,sql语句后面的分号可写可不写-->
      <select id="findall" resultType="com.xxx.domain.User">
      	select * from user
      </select>
      <!--注意：id一定是对应的方法名，resultType不可缺少，这指明了sql返回的结果封装到哪个类中去-->
  </mapper>
  ```

  

- 综上，环境搭建：
  - 创建Maven工程导入依赖
  - 创建实体类Bean和dao的接口
  - 创建主配置文件SqlMyConfig.xml
  - 创建映射配置文件IUserDao.xml

- 注意事项：
  - 1：创建IUserDao.xml和IUserDao.java时，名称是为了和我们之前的知识保持一致，在mybais中它把持久层的接口名称和映射文件也叫做Mapper，所以IUserDao和IUserMapper是一样的。
  - 2：在IDEA中创建目录时，和包不一样
    - 包在创建时 com.xxx.dao是三层
    - 目录创建时 com.xxx.dao是一层
  - 3：mybatis的映射配置文件位置必须和dao接口的包结构相同，也就是说dao接口文件（也叫mapper接口文件）在java下的a.b.c路径下，那么映射配置文件xml也必须在resources下的a.b.c路径下。
  - 4：映射配置文件的mapper标签中 namespace的属性值必须是dao接口的全限定类名
  - 5：映射配置文件的操作配置，id属性的取值必须是dao接口的方法名。
  - 当我们遵从了第3，4，5点之后，我们在开发中就无需再写dao接口的实现类。

- 测试：

  在test/java下创建com.xxx.test包，下面创建测试类（类名自取，无需继承任何东西）

  ```java
  public class Test{
      public static void main(String[] args){
          //1 读取配置文件
          InputStream in = Resources.getResourceAsStream("SqlMyConfig.xml");
          //2 创建SqlSessionFactory工厂
          SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
          SqlSessionFactory factory = builder.build(in);
          //3 使用工厂生产SqlSession对象
          SqlSession session = factory.openSession();
          //4 使用SqlSession创建dao接口的代理对象
          IUserDao userDao = session.getMapper(IUserDao.class);
          //5 使用代理对象执行方法
          List<User> users = userDao.findall();
          for(User user : useers){
              System.out.println(user);//重写该类的toString方法，可以输出这个对象实例
          }
          //6 释放资源
          session.close();
          in.close();
      }
  }
  ```

  - 运行该类的main方法即可。

入门案例：

- 读取配置文件
- 创建SqlSessionFactory工厂
- 创建SqlSession
- 创建DAO接口的代理对象
- 执行dao中的方法
- 释放资源
- 注意：不要忘记在映射配置中告知mybatis返回结果封装到哪个实体类中，也就是resultType属性不可缺少。

##### 使用注解

需要更改内容：

- ```java
  public interface IUserDao{//用户的持久层接口
      @Select("select * from user")
      List<User> findall(); //查询所有操作
  }
  ```

- ```xml
  <!--头部信息-->
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <!--mybatis的主配置文件-->
  <configuration>
  	<!--配置环境-->
      <environments default="mysql">
      	<environment id="mysql">
          	<transactionManager type="JDBC"><!---配置事务类型-->
              </transactionManager>
              <!--配置数据源，也叫连接池 POOLED表示使用连接池-->
              <dataSource type="POOLED">
              	<!--基本信息-->
                  <property name="driver" value="com.mysql.jdbc.driver" />
                  <property name="url" value="jdbc:mysql://localhost:3306/databasename" />
                  <property name="username" value="root" />
                  <property name="password" value="1234" />
              </dataSource>
          </environment>
      </environments>
      
      <!--指定映射配置文件的位置，也就是每个DAO独立的配置文件-->
      <mappers>
      	<mapper class="com/xxx/dao/IUserDao" />
          <!--注意，此处的路径默认指的是在main/resources/文件夹下的com/xxx/dao/...-->
      </mappers>
  </configuration>
  ```

- 运行测试类的main方法即可。

- 总结：把IUserDao.xml移除，在dao接口的方法上使用@Select注解，并指定sql语句。同时需要在SqlMyConfig.xml中的mapper配置时，使用class属性指定dao接口的全限定类名。

- 明确：我们在实际开发中都是越简练越好，都是采用不写dao实现类的方式，不管是用xml还是注解配置，但是mybatis是支持dao实现类的。

##### 设计模式分析

```java
public static void main(String[] args){
        //1 读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMyConfig.xml");
    //这个路径不应该用相对路径（src/...）也不应该用绝对路径（D://。。。）因为项目部署后是没有这些路径的
    //解决：1.使用类加载器，只能读取类路径的配置文件 2.使用ServletContext对象的getRealPath()方法获取项目运行时的真实路径
    
        //2 创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
    //创建工厂使用了构建者模式，只给构建要求，不看细节
    
        //3 使用工厂生产SqlSession对象
        SqlSession session = factory.openSession();
    //生产SqlSession使用工厂模式，优势：解耦，降低类之间的依赖关系。
    
        //4 使用SqlSession创建dao接口的代理对象
        IUserDao userDao = session.getMapper(IUserDao.class);
    //创建DAO实现类使用了代理模式，优势：不修改源码的基础上对已有方法增强
    
        //5 使用代理对象执行方法
        List<User> users = userDao.findall();
        for(User user : useers){
            System.out.println(user);//重写该类的toString方法，可以输出这个对象实例
        }
        //6 释放资源
        session.close();
        in.close();
    }
```

##### 为什么实体类的属性名要和表内字段名一致？

因为mybatis会通过**反射**将sql的执行返回结果封装到实体类中。

### 第二天

1. crud（基于代理dao的方式）
2. 参数深入及结果集深入
3. crud：基于传统dao的方式——编写dao的实现类（了解）
4. mybatis中的配置 (主配置文件: SqlMapConfig.xml）
   - properties标签
   - typeAliases标签
   - mapper标签

#### 1 CRUD操作

- 和上述过程一样，创建maven工程

- 编写数据库表对应的实体类（Bean）

- 编写config.xml配置文件

- 编写DAO接口，定义一个方法（不用实现）（在main/java下com.xxx.dao）

- 编写mapper.xml配置文件（注意路径一定是在main/resources下的，路径名与dao接口必须一致，即com.xxx.dao）

  - namespace和id属性可以指明该mapper对应哪个类中的哪个方法
  - 注意resultType属性不能忘记设置

- 在test/java/下创建路径存放测试类(路径可以是com.xxx.test) （加了@Test注解之后就可以直接运行方法，运行成功！）

  ```java
  package com.xxx.test;
  import org.junit.Test
  public class MybatisTest{
  	
  	@Test
  	public void testFindall(){
  	//1 读取配置文件,生成字节输入流
  	InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
  	//2 获取Factory
  	SqlSessionFactory = new SqlSessionFactoryBuiler().build(in);
  	//3 获取session
  	SqlSession session = factory.openSession();
  	//4 获取dao代理对象
  	IUserDao dao= session.getMapper(IUserDao.class);
  	//5 执行
  	List<User> users = dao.findall();
  	for (User user: users){
  	System.out.println(user);
  	}
  	//6 释放资源
  	session.close();
  	in.close();
  	}
  }
  ```

- 以上是第一天的内容。

- 在dao接口中定义新方法（保存用户的方法）

  ```java
  public interface IUserDao{
      List<User> findall();
      
      void saveUser(User user);//新增
      
      void updateUser(User user);
      
      void deleteUser(Integer userId);//根据id删除用户
      
      User findById(Integer userId);
      
      //根据名称模糊查询
      List<User> findByName(String username);
      
      int findTotal();//查询总用户数
  }
  ```

- 在映射配置的*.xml中添加

  ```xml
  <mapper namespace="com.xxx.dao.IUserDao">
      <!--保存用户-->
      <insert id="saveUser" parameterType="com.xxx.domain.User"><!--参数类型，指明dao接口中对应方法传入的参数类型-->
          <!--配置插入操作后，获取插入数据的id
  		参数有四个：1.属性名称；2 表中的字段名称 3 返回类型 4 是在什么时候执行这个语句，AFTER表示是在insert执行之后
  		-->
          <selectKey keyProperty="id" keyColumn="id" resultType="int" order="AFTER">
          	select lats_insert_id();
          </selectKey>
          
      	insert into user(username) values(#{username});<!--此处的参数名称要和实体类中属性名，也就是数据库表中的字段名，一致，无论是int或date或string类型的参数，都是这种#{名字}的写法-->
      </insert>
      
      <!--更新用户-->
      <update id="updateUser" parameterType="com.xxx.domain.User">
      	update user set username=#{username} where id=#{id};
      </update>
      
      <delete id="deleteUser" parameterType="Integer"> <!--此处参数类型可以是int或java.lang.Integer-->
      delete from user where id=#{uid}; <!--由于对应的方法只传入一个参数，此处不必要求名称一致，只需要随便一个名称用来占位即可-->
      </delete>
      
      <!--根据id查询-->
      <select id="findById" parameterType="INT" resultType="com.xxx.doamin.User">
      select * from user where id =#{uid};
      </select>
      
      <!--根据名称模糊查询-->
      <select id="findByName" parameterType="string" resultType="com.xxx.doamin.User">
      	select * from user where username like #{name};
          <!--此句也可以写成：select * from user where username like '${value}'; 但是注意一定要使用value这个单词-->
      </select>
      
      <!--获取用户总记录条数-->
      <select id="findTotal" resultType="Integer">	
      	select count(id) from user;
      </select>
  </mapper>
  ```

- 在测试类（test文件夹下）中新建一个测试方法

  ```java
  package com.xxx.test;
  import org.junit.Test
  public class MybatisTest{
  	private InputStream in;
  	private SqlSession session;
  	private IUserDao dao;
  	
  	@Before   //用于在测试方法执行之前执行
  	public void init(){
  	//1 读取配置文件,生成字节输入流
  	in = Resources.getResourceAsStream("SqlMapConfig.xml");
  	//2 获取Factory
  	SqlSessionFactory = new SqlSessionFactoryBuiler().build(in);
  	//3 获取session
  	session = factory.openSession(); //这个函数可以传入一个布尔型参数 表示是否设置自动提交，如果传入了true，那么接下来的crud操作既不需要在最后写session.commit()来提交事务了。
  	//4 获取dao代理对象
  	dao= session.getMapper(IUserDao.class);
  	}
  	
  	@After   //用于在测试方法执行之后执行
  	public void destory(){
  	//6 释放资源
  	session.close();
  	in.close();
  	}
  	
  	@Test
  	public void testFindall(){
  	
  	//5 执行
  	List<User> users = dao.findall();
  	for (User user: users){
  	System.out.println(user);
  	}
  	
  
  	
      @Test
      public void testSave(){
      User user = new User();
      user.setId(101);
      user.setUsername("william");//如果字段是日期类型的，那么就需要user.setBirth(new Date())
  	
  	System.out.println("保存之前："+user);
  	//5 执行
  	dao.saveUser(user);//此处与上一个方法相比有不同
  	System.out.println("保存之后："+user); //注意我们在映射xml文件中对insert语句进行了设置，会返回插入的记录的id。这两个println可以看出插入前后，user对象内id属性的变化：从null变为一个具体数字。
  	
  	//提交事务，不加这句的话，仅凭以上代码，运行后事务会回滚
  	session.commit();
  	}
  	@Test
      public void testUpdate(){
      User user = new User();
      user.setId(101);
      user.setUsername("william");//如果字段是日期类型的，那么就需要user.setBirth(new Date())
  	
  	//5 执行
  	dao.updateUser(user);
  	
  	//提交事务，不加这句的话，仅凭以上代码，运行后事务会回滚
  	session.commit();
  	
  	}
  	@Test
      public void testDelete(){
      
  	//5 执行
  	dao.deleteUser(101);
  	
  	//提交事务，不加这句的话，仅凭以上代码，运行后事务会回滚
  	session.commit();
  	}
  	
  	@Test
    public void testFindById(){
  	//5 执行
  	User use = dao.findById(101);//
  	//提交事务，不加这句的话，仅凭以上代码，运行后事务会回滚
  	session.commit();
  	}
  	
  	@Test
      public void testFIndByName(){
  	//5 执行
  	List<User> use = dao.findByName("%王%");//
  	//提交事务，不加这句的话，仅凭以上代码，运行后事务会回滚
  	session.commit();
  	}
  	
  	@Test
  	public void testFindTotal(){
  	//5 执行
  	int num = dao.findTotal();
  
  }///注意，session.commit()方法（事务提交）可以用在任何操作后面，查询操作不加这句也可以，但是保存操作必须加。
  ```
  
   

##### **#{}和${}的区别？**



#### 2 深入parameterType

1. 传递简单类型

2. 传递pojo对象（就是数据库表的实体类对象）

   mybatis使用ognl表达式解析对象字段的值，#{}或${}中的值为pojo的属性名称。

3. 传递pojo包装对象（就是写一个类，这个类里包含多个属性，每个属性又是某一个实体类的实例，这样，这个类里面就包含不止一个数据库表的实体类信息）

   开发中通过pojo传递查询条件，查询条件是综合的查询条件，不仅包括用户自身某些信息做查询条件还包括其他查询条件（比如另一张数据库表中的用户订单信息），这时可以使用包装对象传递输入参数，pojo类中包含pojo。

- 在IUserDao.java（dao接口）中写

  ```java
  package com.xxx.dao;
  public interface IUserDao{
      //根据QueryVp中的查询条件查询用户
   	List<User> findUserByVo(QueryVo vo);   
  }
  ```

- 在com.xxx.doamin中写

  ```java
  package com.xxx.domain;
  public class QueryVo{
      private User user;
      public User getUser(){
          return User;
      }
      public void setUser(User user){
          this.user=user;
      }
  }
  ```

- 在IUserDao.xml（映射配置xml，位于resources下的com.xxx.dao,必须与dao接口相同路径）中写

  ```xml
  <mapper>
  	<!--根据QueryVo的条件查询用户-->
      <select id="findUserByVo" parameterType="com.xxx.domain.QueryVo" resultType="com.xxx.domain.User">
          select * from user where username like #{user.username}
          <!--注意，这就是ognl表达式的用法，user是parameterType中指出的QueryVo类的属性名，而username是user的属性名，用英文点直接连接-->
      </select>
  </mapper>
  ```

- 测试类的写法

  ```java
  	@Test
      public void testFIndByVo(){
          QueryVo vo = new QueryVo();
          User user = new User();
          user.setUsername("%王%");
          vo.setUser(user);
  	//5 执行
  	List<User> users = dao.findUserByVo(vo);//
  	//提交事务，不加这句的话，仅凭以上代码，运行后事务会回滚
  	session.commit();
  	}
  ```

  

#### 3 深入resultType

返回类型有：

1. 普通类型（int，Integer String）
2. pojo对象
3. pojo对象的列表

- 如果实体类中属性名和数据库表的字段名不一致？

  执行sql语句时，传入的参数可以根据实体类中的属性名修改（包括测试类的方法中的参数，以及映射配置xml文件中#{}中的参数名），因此传入参数方面是没有影响的。

  但是在sql语句返回时，需要把返回值封装到pojo对象里，如果表的字段名和pojo对象的属性名不一致就封装不进去。（在win系统中，mysql不区分大小写，因此如果pojo类中属性名和表字段名只是大小写不一致的话，也可以封装。linux系统中mysql区分大小写）

- 那么如何解决实体类中属性名和数据库表的字段名不一致，导致返回结果无法封装的问题呢？

  - 1.要么严格遵守实体类属性名和数据库表字段名一致

  - 2.要么在xml中给返回的字段起别名

    ```xml
    <select id="findall" resultType="com.xxx.domain.User">
    	select id as uid, username as uName from user;
    </select>
    <!--其中id，username为字段名，uid，uName为对应的属性名，这样使他们匹配上就可以封装了。as可以省略，用空格代替-->
    ```

  - 3.要么在xml中配置一下对应关系(开发效率快，但执行效率不如方式2)

    ```xml
    <!--配置，使列名和实体类属性名对应-->
    <mapper>
        <resultMap id="userMap" type="com.xxx.domain.User">
            <!--主键字段的对应-->
            <id property="uid" column="id"></id>  <!--id为字段名，uid为属性名-->
            <!--非主键字段的对应-->
            <result property="uName" column="username"></result>  <!--username为字段名，uName为属性名-->
        </resultMap>
    
        <select id="findall" resultMap="userMap"> 
            <!--注意：使用了resultMap之后，此处不再使用resultType，而是使用resultMap=“resultMap的id值”-->
            select id, username as from user;
        </select>
    </mapper>
    ```

#### 4 property标签

- main/resources目录下的SqlMyConfig.xml，关于数据库的连接配置部分可以换一种写法，使用properties标签：

```xml
<!--头部信息-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--mybatis的主配置文件-->
<configuration>
    
    <!--配置properties
	可以在标签内部配置数据库连接的信息，也可以通过属性引用外部配置文件信息（比如*.properties文件）	
	resource属性，常用
		用于指定配置文件的位置，是按照类路径的写法来写（即com.xxx.xxxx.*.properties）,而且必须存在于类路径下
	url属性
		按照url的写法来写地址
		URL：统一资源定位符，可以唯一标识一个资源的位置
		写法：
			http://localhost:8080/mybatisserver/demo
			协议	     主机    端口  URI
		URI：统一资源标识符，可以在应用中唯一定位一个资源。
	-->
    <properties>
    	<property name="driver" value="com.mysql.jdbc.driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/databasename" />
        <property name="username" value="root" />
        <property name="password" value="1234" />
    </properties>
    
    
	<!--配置环境-->
    <environments default="mysql">
    	<environment id="mysql">
        	<transactionManager type="JDBC"><!---配置事务类型-->
            </transactionManager>
            <!--配置数据源，也叫连接池 POOLED表示使用连接池-->
            <dataSource type="POOLED">
            	<!--基本信息--><!--此处的value用properties标签下的name代替-->
                <property name="driver" value=${driver} />
                <property name="url" value=${url} />
                <property name="username" value=${username} />
                <property name="password" value=${password} />
            </dataSource>
        </environment>
    </environments>
    
    <!--指定映射配置文件的位置，也就是每个DAO独立的配置文件-->
    <mappers>
    	<mapper resources="com/xxx/dao/IUserDao.xml" />
        <!--注意，此处的路径默认指的是在main/resources/文件夹下的com/xxx/dao/...-->
    </mappers>
</configuration>
```

- 还有一种写法，是把配置写在*.properties文件中，然后在xml中读取配置信息：

  - 比如：在resources下创建jdbcConfig.properties文件

    ```properties
    jdbc.driver=com.mysql.jdbc.Driver
    jdbc.url=jdbc:mysql://localhost:3306/databasename
    jdbc.username=root
    jdbc.password=123456
    ```

  - 然后修改xml文件

    ```xml
    <!--头部信息-->
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <!--mybatis的主配置文件-->
    <configuration>
        
        <!--配置properties
    	由于jdbcConfig.properties就在resources目录下，所以resource属性值按下面的写法，如果这个properties文件在resources路径下的com.xxx.property文件夹下，就要写成
     resource="com.xxx.property。jdbcConfig.properties"。
    	-->
    	
        <properties resource="jdbcConfig.properties">
        </properties>
        
        
    	<!--配置环境-->
        <environments default="mysql">
        	<environment id="mysql">
            	<transactionManager type="JDBC"><!---配置事务类型-->
                </transactionManager>
                <!--配置数据源，也叫连接池 POOLED表示使用连接池-->
                <dataSource type="POOLED">
                	<!--基本信息--><!--注意：此处的value用properties文件里的名称-->
                    <property name="driver" value=${jdbcdriver} />
                    <property name="url" value=${jdbc.url} />
                    <property name="username" value=${jdbc.username} />
                    <property name="password" value=${jdbc.password} />
                </dataSource>
            </environment>
        </environments>
        
        <!--指定映射配置文件的位置，也就是每个DAO独立的配置文件-->
        <mappers>
        	<mapper resources="com/xxx/dao/IUserDao.xml" />
            <!--注意，此处的路径默认指的是在main/resources/文件夹下的com/xxx/dao/...-->
        </mappers>
    </configuration>
    ```

#### 5 typeAliases标签和package标签

```xml
<!--头部信息-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--mybatis的主配置文件-->
<configuration>
 
    <!--配置properties-->
    <properties resource="jdbcConfig.properties">
    </properties>
    
 	<!--使用typeAliases配置别名-->
    <typeAliases>
        <!--1。  typeAlias属性配置别名
		type属性是实体类全限定类名，alias是别名，配置之后，映射配置文件（也就是IUserDao.xml，写sql语句的那个）中就可以在任何位置 使用别名代替全限定类名的实体类，而且别名不缺分大小写，即使用时写成USER和user都成。-->
    	<!-- <typeAlias type="com.xxx.domain.User" alias="user"></typeAlias>  -->
        
        <!--2. package属性配置别名  
		name属性指定包路径，该路径下所有实体类都会注册别名，且别名就是类名，不缺分大小写-->
        <package name="com.xxx.domain"></package>
    </typeAliases>
    
	<!--配置环境-->
    <environments default="mysql">
    	<environment id="mysql">
        	<transactionManager type="JDBC"><!---配置事务类型-->
            </transactionManager>
            <!--配置数据源，也叫连接池 POOLED表示使用连接池-->
            <dataSource type="POOLED">
            	<!--基本信息-->
                <property name="driver" value=${jdbcdriver} />
                <property name="url" value=${jdbc.url} />
                <property name="username" value=${jdbc.username} />
                <property name="password" value=${jdbc.password} />
            </dataSource>
        </environment>
    </environments>
    
    <!--指定映射配置文件的位置，也就是每个DAO独立的配置文件-->
    <mappers>
    	<!-- <mapper resources="com/xxx/dao/IUserDao.xml" />  -->
        <!--这个mappers标签下，也有package标签 name属性是指明dao接口所在的包（应该是映射配置文件所在的包吧。当然，二者路径是一致的），这样就不必用再写mapper，resources或class了-->
        <package name="com.xxx.dao"></package>
        
    </mappers>
</configuration>
```

### 第三天 

1. 连接池与事务控制 （会用，原理了解）
   - 连接池使用及分析
   - 事务控制分析
2. 基于xml配置的动态sql语句使用（会用）
   - mappers配置文件中的几个标签<if> <where> <foreach> <sql>
3. 多表操作（掌握）
   - 一对一（？）
   - 一对多
   - 多对多

#### 1 mybatis中的连接池

mybatis提供了三种方式配置

​	配置的位置：主配置文件SqlMyConfig.xml中的dataSource标签，type属性就是表示采用何种连接池方式

​	type的取值：

		- POOLED  采用传统的javax.sq.DataSource规范中的连接池，mybatis中有针对规范的实现
		- UNPOOLED 采用传统的获取连接的方式，虽然也实现javax.sql.DataSource接口但是并没有使用池的思想，只是创建连接来使用，使用后关闭连接
		- JNDI 采用服务器提供的JNDI技术实现，来获取DataSource对象，不同服务器所能拿到的DataSource是不一样的。注意：如果不是web或maven的war工程则不能使用。

tomcat服务器，采用连接池就是dbcp连接池。

#### 2 mybatis中的事务

- 什么是事务
- 四大特性ACID
- 不隔离产生的三个问题
- 解决办法：4种隔离级别



它本质上（分析源码可知）还是通过sqlsession对象的commit和rollback方法实现提交和回滚。



##### 1 动态sql if标签

在IUserDao.java接口新增一个方法：

```java
public List<User> findByCondition(User user);
```

在IUserDao.xml中添加新增方法的sql。

```xml
<mapper>
	<!--根据条件查询-->
    <select id="findByCondition" resultMap="userMap" parameterTyupe="user"> <!--user是com.xxx.domain.User配置后的别名-->
    	select * from user where 1=1
        <if test="uName!=null">
        	and username = #{uName}
        </if>
        <if test="uid!=null">
        	and id = #{uid}
        </if>
        <!--注意此处if标签的妙用，它是先判断传入的user参数的属性uName是否为空，如果不为空，就在sql中补充上一条查询条件-->
    </select>
</mapper>
```

完成！

##### 2 动态sql where标签

在上述的IUserDao.xml中修改一下：使用where标签代替“where 1=1”

```xml
<mapper>
	<!--根据条件查询-->
    <select id="findByCondition" resultMap="userMap" parameterTyupe="user"> <!--user是com.xxx.domain.User配置后的别名-->
    	select * from user 
        <where>
            <if test="uName!=null">
                and username = #{uName}
            </if>
            <if test="uid!=null">
                and id = #{uid}
            </if>
        </where>
        <!--注意此处where标签的妙用-->
    </select>
</mapper>
```

##### 3 动态sql foreach标签

需求：select * from user where id in(1,2,4,8);

```java
package com.xxx.domain;
public class QueryVo{
    private User user;
    private List<Integer> ids;
    public  User getUser(){
        return user;
    }
    public void setUser(User user){
        this.user=user;
    }
    public List<Integer> getIds(){
        return ids;
    }
    public void setIds(List<Integer> ids){
        this.ids=ids;
    }
    
}
```

在IUserDao.java接口中新增方法

```java
/*
根据QueryVo中提供的id集合查询用户
*/
List<User> findUserInIds(QueryVo vo);
```

在IUserDao.xml中新增

```xml
<mapper>
	<!--根据-->
    <select id = "findUserInIds" resultMap="userMap" parameterType="queryvo">
    	select * from user 
        <where>
        	<if test="ids!=null and ids.size()>0">
                <foreach collections="ids" open="and id in (" close=")" item="xxxid" separator=",">
                #{xxxid}  <!--xxxid表示名称自取-->
                </foreach>
            </if>
        </where>where id in
    </select>
</mapper>
```

在测试类中新增一个测试方法

```java
@Test
public void testFindInIds(){
    QueryVo vo = new QueryVo();
    List<Integer> list = new ArrayList<>();
    list.add(41);
    list.add(42);
    list.add(46);
    
    List<User> users = userDao.findUserInIds(vo);
    for (User u : users){
        System.out.println(u);
    }
}
```

##### 4 动态sql  sql标签

xml文件中，可以将一些重复的sql语句定义一下。sql标签与select标签同级。

```xml
<sql id="xxxx">
    select * from user;
</sql>

<select id="findall" resultMap="userMap">
    <include refid="xxxx"></include>
</select>
```

### 3 多表查询

#### 1 表之间的关系

一对多 多对一 一对一 多对多

例子：

- 用户和订单就是一对多
- 订单和用户就是多对一
  - 一个用户可以下多个订单
  - 多个订单属于同一用户
- 人和身份证号就是一对一
- 老师和学生就是多对多
  - 一个学生可以有多个老师
  - 一个老师可以教多个学生

特例：如果拿出每一个订单，他都只属于一个用户

所以mybatis就把多对一看成了一对一。

#### 2 开始多表查询

示例：

1. 数据库中建立两个表，用户表，账户表，一个用户有多个账户，一个账户只能属于一个用户，账户表(account)的字段为id，uid（外键，用户表的id），money。用户表(user)的字段为id，username, address,birthday,sex。
2. 建立两个实体类：用户和账户实体类，实体类要体现出一对多的关系
3. 建立两个配置文件（用户和账户的xml映射配置文件，主配置文件一个即可）
4. 实现配置



- 按照之前的过程建一个maven项目

- 建立account数据库表的实体类，com.xxx.domain.Account

  ```java
  package com.xxx.domain;
  public class Account implements Serializable{
      private Integer id;
      private Integer uid;
      private Double money;//数据库中该字段的类型是Double 
      
      //生成get set方法，此处省略
      //可以重写toString方法，这样可以打印实例对象
  }
  ```

- 创建com.xxx.dao.IAccountDao接口

  ```java
  public interface IAccountDao{
      
      List<Account> findAll();//查询所有账户
      
  }
  ```

  

- 在resources文件夹下创建com.xxx.dao.IAccountDao.xml

  ```xml
  头部信息
  <mapper namespace="com.xxx.dao.IAccountDao">
      <select id="findAll" resultType="com.xxx.domain.Account">
      select * from account
      </select>
  </mapper>
  ```

  

- 在test文件夹下的java文件夹下创建测试类com.xxx.test.AccountTest.java,内容和之前写的测试类基本类似。

  ```java
  @Test
  public void testFindAll(){
      List<Account> accounts = accountDao.findAll();
      for (Account account:accounts){
          System.out.println(account);
      }
  }
  ```

  运行该测试方法，成功！

  

#### 新的需求：

- ```java
  public interface IAccountDao{
      
      List<Account> findAll();//查询所有账户
      
      List<AccountUser> findAllAccount();//查询所有账户，并且带有所属用户的username 和address信息
      
  }
  ```

- 新建一个实体类com.xxx.domain.AccountUser.java，这个实体类是这样的

  ```java
  public class AccountUser extends Account{
      private String username;
      private String address;
      
      //生成get set方法，此处省略
  }
  ```

- 在IAccountDao.xml中添加

  ```xml
  <mapper namespace="com.xxx.dao.IAccountDao">
      <select id="findAll" resultType="com.xxx.domain.Account">
      select * from account
      </select>
      <!--查询所有账户，并且带有所属用户的username 和address-->
      <select id="findAllAccount" resultType="com.xxx.domain.AccountUser">
  		select a.*,u.username, u.address from account a, user u where u.id=a.id;
      </select>
  </mapper>
  
  ```

- 测试类新增测试方法

  ```java
  @Test
  public void testFindAllAccountUser(){
      List<AccountUser> aus = accountDao.findAllAccount();
      for (AccountUser au:aus){
          System.out.println(au);
      }
  }
  ```

#### 常用方式：

以上方式并不常用，常用方式是建立两个表各自的实体类，而且需要在实体类中体现出一对多的关系。

如何做呢？

答：在Account的实体类中，除了自身的属性对应自己的各个字段之外，还需要加一个User类的属性，表示该账户属于哪个用户。

- ```java
  public Account implements Serializable{
      private User user;//从表实体类应该包含一个主表实体类的对象引用
      private Integer id;
      private Integer uid;
      private Double money;
      //对应的get set方法需要添加
  }
  ```

- IAccountDao.xml

  ```xml
  <mapper>
  	<!--定义封装account和user的resultMap-->
      <resultMap id="accountUserMap" type="com.xxx.domain.Account">
          <id property="id" column="aid"></id>
          <result property="uid" column="uid"></result>
          <result property="money" column="money"></result>
          <!--一对一的关系映射 配置封装user的内容-->
          <assocaition property="user" column="uid" javaType="com.xxx.domain.User">
              <!--javaType表示封装到哪个对象-->
          	<id property="id" column="id"></id>
          	<result property="username" column="username"></result>
          	<result property="address" column="address"></result>
          	<result property="sex" column="sex"></result>
          	<result property="birthday" column="birthday"></result>
          </assocaition>
      </resultMap>
      <!--查询所有，此处与之前的写法有更改，因为在account类中新增了User对象做属性-->
      <select id="findAll" resultMap="accountUserMap">
      	select u.*, a.id as aid, a.uid,a.money from account a ,user u where u.id=a.id
      </select>
  </mapper>
  ```

#### 3 一对多查询

#### 需求：

查询用户时附带查询该用户的所有账户信息

- IUserDao.java接口中

```java
public interface IUserDao{
    
    //查询用户时附带查询该用户的所有账户信息
	public List<User> findAll();

}
```

- User实体类中

  ```java
  package com.xxx.domain;
  public class User implements Serializable{
      private Integer id;
      private String username;
      private String sex;
      private String address;
      private Date birthday;
      private List<Account> accounts;//主类中应包含从类的集合引用
      
      //以下添加上get set 方法，此处省略
  }
  ```

- 测试类

  ```java
  public class UserTest{
      private InputStream in;
      private SqlSession sqlSession;
      private IUserDao userDao;
      
      //init和destory方法此处省略
      
      @Test
      public void testFindAll(){
          List<User> users = userDao.findAll();
          for(User u:users){
              System.out.println(u);
              System.out.println(u.getAccounts());
          }
      }
      
  }
  ```

- IUserDao.xml

  ```xml
  头部信息
  <mapper namespace="com.xxx.dao.IUserDao">
      <!--定义User的resultMap-->
      <resultMap id="userAccountMap" type="com.xxx.domain.User">
          <id property="id" column="id"></id>
          <result property="username" column="username"></result>
          <result property="address" column="address"></result>
          <result property="sex" column="sex"></result>
          <result property="birthday" column="birthday"></result>
          <!--配置User对象中accounts集合的映射-->
          <collection  property="accounts" ofType="com.xxx.domain.Account">
          	<id property="aid" column="id"></id>
              <result property="uid" column="uid"></result>
              <result property="money" column="money"></result>
          </collection>
      </resultMap>
      
      <select id="">
      	select * from user u left outer join account a on u.id=a.id
      </select>
  </mapper>
  ```

#### 4 多对多查询

示例：用户和角色（角色就是身份，一个用户有多个角色，一个角色也可以赋予多个用户，二者是多对多的关系）

1. 数据库建立两张表（用户，角色，让其具有多对多的关系，使用中间表，中间表中包含各自的主键，在中间表中使用两个外键分别连接两个表的主键）
   - role表：id， role_name， role_desc
   - user表：id，username,sex,address,birthday
   - user_role(中间)表：id，uid，rid
2. 建立两个实体类（User用户实体和Role角色实体，让他们体现出多对多的关系：各自包含一个对方的集合引用）
3. 建立各自的接口文件IUserDao.java  IRoleDao.java
4. 建立两个配置文件IUserDao.xml  IRoleDao.xml
5. 实现配置
   - 查询用户时，可以得到该用户所包含的所有角色的信息
   - 查询角色时，可以得到该角色所赋予的用户信息

- 在com.xxx.domain下创建role的实体类

  ```java
  package com.xxx.domain;
  public class Role implements Serializable｛
  	
  	private Integer roleId;//此处没有和字段名对应，而是按照java规范
  	private String roleName;
  	private String roleDesc;
  	private List<User> users;//一个角色可以赋予给多个用户
          
      //生成get set方法
  
  ｝
  ```

- dao接口

  ```java
  package com.xxx.dao;
  public interface IRoleDao{
      List<ROle> findAll();//查询所有结果
  }
  ```

- 在resources下创建xml文件：com.xxx.dao.IRoleDao.xml

  ```xml
  头部信息
  <mapper namespace="com.xxx.dao.IRoleDao">
      <!--定义role的resultMap-->
      <resultMap id="roleMap" type="com.xxx.domain.Role">
          <id property="roleId" column="rid"></id>
          <result property="roleName" column="role_name"></result>
          <result property="roleDesc" column="role_desc"></result>
          <collection property="users" ofType="com.xxx.domain.User">
              <id property="id" column="id"></id>
              <result property="username" column="username"></result>
          	<result property="address" column="address"></result>
              <result property="sex" column="sex"></result>
          	<result property="birthday" column="birthday"></result>
          </collection>
      </resultMap>
      
      <select id="findAll" resultMap="roleMap">
          select u.*,r.id as rid, r.role_name, r.role_desc from role r
          left outer join user_role ur on r.id=ur.rid<!--可以换行，但是本行首部或上行尾必须加个空格，避免拼接成 rleft -->
          left outer join user u on u.id=ur.uid
          
      </select>
  </mapper>
  ```

- 写一个Role的测试类，和之前写法一致：

  ```java
  private IRoleDao roleDao;
  
  @Test
  public void testFindAll(){
      List<Role> roles = roleDao.findAll();
      for(Role r:roles){
          System.out.println(r);
          System.out.println(r.getUsers());
      }
  }
  ```

##### 接下来查询所有用户并附带该用户所有的角色信息

只需要改一下sql

```sql
select u.*,r.id as rid, r.role_name, r.role_desc from user u
        left outer join user_role ur on u.id=ur.uid
        left outer join role r on r.id=ur.rid
```

具体做法：

- 用户实体类User中修改

  ```java
  private List<Role> roles;
  //生成get set 方法
  ```

- IUserDao.xml

  ```xml
  <resultMap id="userMap" type="com.xxx.domain.User">
     	<id property="id" column="id"></id>
          <result property="username" column="username"></result>
          <result property="address" column="address"></result>
          <result property="sex" column="sex"></result>
          <result property="birthday" column="birthday"></result>
          <!--配置User对象中角色集合的映射-->
          <collection  property="roles" ofType="com.xxx.domain.Role">
          	<id property="roleId" column="rid"></id>
              <result property="rolenName" column="role_name"></result>
              <result property="roleDesc" column="role_desc"></result>
          </collection>
  </resultMap>
  <select id ="findAll" resultMap="userMap">
  	select u.*,r.id as rid, r.role_name, r.role_desc from user u
          left outer join user_role ur on u.id=ur.uid
          left outer join role r on r.id=ur.rid
  </select>
  ```

- 修改测试类

  ```java
  @Test
  public void findAll(){
      List<User> users = userDao.findAll();
      for (User u:users){
          System.out.println(u);
          System.out.println(u.getRoles());
      }
  }
  ```

  





