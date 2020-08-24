# 整合JDBC

对于数据访问层，无论是关系型或菲关系型数据库，springboot都是采用spring data进行统一处理。

- 创建项目，加入依赖：JDBC API；MySQL driver（这是整合JDBC相关的两个依赖），在pom中分别是spring-boot-starter-jdbc和mysql-connector-java。

- resources/application.yml:

  ```yml
  spring:
  	datasource:
  		username: root
  		password: 123456
  		url: jdbc:mysql://localhost:3306/datasourcename?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC # 必须加这个时区参数 防止时区报错
  		driver-class-name: com.mysql.jdbc.driver # 数据库版本5以前的用这个 8以后的用com.mysql.cj.jdbc.driver
  		
  ```

- 测试类：

  ```java
  package com.kuang;
  
  @SpringBootTest
  class Tests{
      
      @Autowired
      DataSouce dataSource;//进行了上述配置后spring中就会有这个bean
      
      @Test
      void contextLoads(){
          //查看默认数据源  hikari
          System.out.println(dataSource.getClass());
          //获取连接
          Connection conn = dataSource.getConnection();
          System.out.println(conn);
          conn.close();
          
         
      }
  }
  ```

- com.kuang.controller

  ```java
  package com.kuang.controller;
  
  @RestController  //这个是需要web依赖
  public class JDBCController{
       //springboot中有很多xxxTemplate是已经配置好的模板bean,可以直接拿来用
          
      @Autowired
      JdbcTemplate jdbcTemplate;
      
      //查询数据库所有信息
      //没有实体类，数据库中的数据如何获取 Map
      
      @GetMapping("/userList")
      public List<Map<String, Object>> userList(){
          String sql = "select * from user";
          List<Map<String, Object>> maps = jdbcTemplate.queryForList(sql);
          return maps;//给前端
      }
  //去浏览器尝试发起请求，会发现maps的内容出现在浏览器上（一个[] 里面是 {"字段":value}）
  	@GetMapping("/addUser")
      public String addUser(){
          String sql = "insert into user (id,name,pwd) values(4,'小米','123')";
          //Spring sql = "update user set name=?,pwd=? where id = "+id
          jdbcTemplate.update(sql);
          return "ok";
      }
      //spring设置了自动提交。
      
      @GetMapping("/updateUser/{id}")
      public String updateUser(@PathVariable("id") int id){
          //String sql = "insert into user (id,name,pwd) values(4,'小米','123')";
          Spring sql = "update user set name=?,pwd=? where id = "+id;
          //封装
          Object[] objs = new Object[2];
          objs[0] = "xoaom";
          objs[1] = "zzz";    
          jdbcTemplate.update(sql,objs);//一种传参的方式
          return "ok";
      }
      
      @GetMapping("/deleteUser")
      public String deleteUser(@PathVariable("id") int id){
          Spring sql = "delete from user where id =?";
          jdbcTemplate.update(sql,id); //id是int型，也是一个object类型
          return "ok";
      }
      
  }
  
  ```

  

# 整合数据源Druid

Druid的特点是自带监控功能

Hikari（springboot默认数据源）特点是快

整合Druid只需要**在上面的基础上**：

- pom文件中导入druid依赖

- 修改yml文件

  ```yml
  spring:
  	datasource:
  		username: root
  		password: 123456
  		url: jdbc:mysql://localhost:3306/datasourcename?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC # 必须加这个时区参数 防止时区报错
  		driver-class-name: com.mysql.jdbc.driver # 数据库版本5以前的用这个 8以后的用com.mysql.cj.jdbc.driver
  		type: com.alibaba.druid.pool.DruidDataSource
  ```

- 完成，就这么简单！

关于druid数据源的 一些私有配置，springboot默认是不导入以下这些配置的，需要自己绑定。

```yml
spring:
	datasource:
		# 常见参数
		initialSize: 
		minIdle: 
		maxActive:
		maxWait:
		...
		
		# Druid比其他牛逼的一些地方的配置
		filters: stat,wall,log4j # 配置监控统计拦截的filters：stat：监控统计；log4j：日志记录；wall：防御sql注入(使用log4j的话需要先导入log4j依赖到pom中)
		maxPoolPreparedStatementPerConnectionSize: 20
		useGlobalDataSourceStat: true
		connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```



```java
/*绑定yml中的配置*/
package com.kuang.config;
@Configuration
public class DruidCOnfig{
    @ConfigurationProperties(prefix="spring.datasource")
    @Bean
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }
    //这样，返回的这个new出来的bean就与yml中的配置绑定了
    
    //后台监控（页面）
    //因为springboot内置了servlet容器，所以没有web.xml,替代方法：ServletRegistrationBean；想注册什么就new什么即可
    @Bean
    public ServletRegistrationBean statViewServlet(){
         ServletRegistrationBean<StatViewServlet> bean=new ServletRegistrationBean<>(new StatViewServlet(),"/druid/*");//只要访问这个路径就会进入后台登录页面
        
        //后台需要有人登录， 配置账号密码
        HashMap<String,String> initParameters = new HashMap<>();
        //增加配置
        initParameters.put("loginUsername","admin");//登录key 是固定的loginUsername，loginPassword。
        initParameters.put("loginPassword","123");
        //允许谁访问
        initParameters.put("allow","");
        //禁止谁访问
        initParameters.put("kuang","192.168.x.xx");
        bean.setInitParameters(initParameters);//设置初始化参数
        return bean;
        
    }
    
    //过滤器
    @Bean
    public FilterRegistrationBean webStatFilter(){
        new FilterRegistrationBean ben = new FilterRegistrationBean();
        bean.setFilter(new webStatFIlter());
        //可以过滤哪些请求呢？
        Map<String ,String> initParameters = new HashMap<>();
        //这些东西不进行统计
        initParameters.put("exclusion","*.js,*.css,/druid/*");
        
        return bean;
    }
    
}
```





### jdbc是建立在各种特定的数据库驱动之上的数据库驱动，还是一种数据源？

# 整合Mybatis

- 新建项目，添加web依赖，jdbc依赖，mysql驱动依赖

- 在pom文件中添加mybatis依赖：mybatis-spring-boot-starter

- 配置数据库信息：application.yml

  ```yml
  
  spring:
  	datasource:
  		username: root
  		password: 123456
  		url: jdbc:mysql://localhost:3306/databasename?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
  		driver-class-name: com.mysql.cj.jdbc.driver
  ```

- 在test/java下建测试类

  ```java
  package com.kuang;
  @SpringBootTest
  class Tests{
      
      @Autowired
      DataSource dataSource;
      
      @Test
      void contextLoads(){
          System.out.println(dataSource.getClass());
          System.out.println(dataSource.getConnection());
          
      }
  }
  ```

- pom中导入jar包依赖[lombok](https://www.cnblogs.com/heyonggang/p/8638374.html)（用于简化代码，通过注解来自动生成getter setter等方法）

- 写一个pojo

  ```java
  package com.kuang.pojo;
  
  @Data//生成getter setter 重写hashcode toString方法
  @NoArgsConstructor  //无参构造
  @ArgsConstructor   //有参构造
  public class USer{
      private int id;
      private String name;
      private String pwd;
      
  }
  ```

- 写mapper接口类

  ```java
  package com.kuang.mapper;
  @Mapper //这个注解表示这是个mybatis的mapper类  ，如果不加这个，可以在启动类上加一个@MapperScan注解来扫描。以上两种方式选其一即可
  @Repository //这个注解必须有，是dao层注解，也可以用@component这个万能的注解
  public interface userMapper{
      List<User> queryUserList();
      User queryUserById(int id);
      int addUser(User user);
      int updateUser(User user);
      int deleteUser(int id);
  }
  
  ```

- 在resources下创建xml映射文件 mybatis.mapper.UserMapper.xml

  ```xml
  头部信息
  <mapper namespace="com.kuang.mapper.UserMapper">
  	<select id="queryUserList" resultType="User"> # 这里可以使用User是因为下面配置文件中增加了mybatis的配置
          select * from user;
      </select>
      
      <select id="queryUserById" resultType="User">
      	select * from user where id = #{id}
      </select>
      <insert id="addUser" parameterType="User">
      insert into user(id,name,pwd) values(#{id},#{name},#{pwd})
      </insert>
      <update id="updateUser" parameterType="User">
      	update user set name=#{name},pwd=#{pwd} where id=#[id]
      </update>
      <delete id="deleteUser" parameterType="int">
      delete from user where id=#{id}
      </delete>
  </mapper>
  ```

- 在配置文件application.yml中整合mybatis：

  ```yml
  spring:
  	datasource:
  		username: root
  		password: 123456
  		url: jdbc:mysql://localhost:3306/databasename?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
  		driver-class-name: com.mysql.cj.jdbc.driver
  # mybatis配置：
  mybatis:
  	type-aliases-package: com.kuang.pojo
  	mapper-location: classpath:mybatis/mapper/*.xml
  # classpath相当于src/main/resources文件夹，mapper-location是声明xml的位置
  # type-aliases-package是声明pojo的位置，这样在xml中的resultType参数就可以直接使用pojo的类名
  
  ```

- ```java
  package com.kuang.controller;
  @RestController
  public class UserController{
      @Autowired
      private UserMapper userMapper;
      @GetMapping("/queryUserList")
      public List<User> queryUserList(){
          List<User>users = userMapper.queryUserList();
          return users
      }
  }
  ```

- 在浏览器输入localhost：8080/queryUserList就可以看到返回的结果。

MVC：

M：数据和业务

C：交接

V：html等前端



1. 导入包
2. 配置文件
3. mybatis配置
4. 编写sql
5. service层调用dao层
6. controller层调用service层



