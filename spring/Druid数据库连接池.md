## Druid数据库连接池

### 一、原理

+ JDBC

  JDBC，大家经常说JDBC，但是它到底是个什么东西，其实我的理解它是一个规范，让大家对数据库的操作产生一个共识的规范，有了他大家就使用统一的标准去访问数据库，并且这个规范被Java代码实现了，这样产生了一个好处，就是应用程序再操作数据库时，不用再从底层做起，而是站在了JDBC肩膀上。JDBC除了向上为应用程序提供了API规范，同时向下为数据库厂商定义了规范，JDBCDriverApi，就是说，只要你的数据库提供了JDBCDriverApi规范定义的接口，那么JDBC就可以直接连接你的数据库，为大家使用，这就是各大数据库为JDBC提供的驱动程序

+ 数据库连接池

  按照上面说的，数据库有了，并且数据库也实现了JDBCDriver规范，那么我们就可以通过JDBC连接数据库了，确实，有了这两个东西，我们就可以实现功能了。那么数据库连接池是什么呢，是干什么用的呢？其实在JDBC方位数据库的时候是通过连接来完成的，但是连接的打开及关闭非常耗时，这时就是数据库连接池的作用了，他来管理这个数据库连接，不会用完立即关闭，可以缓存起来，供下次使用，为JDBC连接数据库提供锦上添花的作用，Druid就是一个数据库连接池，用来管理数据库连接池的

+ mybatis

  刚才说JDBC已经为应用程序提供了标注的访问数据库的规范已经实现了，那还要mybatis干什么呢？既然是标准，是规范，就只能是定义实现很通用的API，不可能完全考虑易用性，要把这种方面的扩充交给其他人来做，这是mybatis就借助JDBC，在JDBC的基础上封装出了功能更加强大的API，进一步帮助开发人员操作数据库

+ 总结

  ![image-20200305155304343](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200305155304343.png)

  

### 二、搭环境

1. 引入pom，由上图所示，我们需要三个东西，一个是链接数据库的驱动，用来链接数据库（省略），第二个是Druid用来做数据库连接池（如下），第三个是mybatis作为JDBC的升级版本使用（省略）

   ```xml
   <!--不是这个-->	
   <dependency>
   		<groupId>com.alibaba</groupId>
   		<artifactId>druid</artifactId>
   		<version>1.1.10</version>
   	</dependency>
   <!-- 是这个 -->
   <dependency>
               <groupId>com.alibaba</groupId>
               <artifactId>druid-spring-boot-starter</artifactId>
               <version>1.1.10</version>
           </dependency>
   ```

2. 配置数据库

   + 第一种方式

     ```properties
     # 数据库访问配置
     # 主数据源，默认的
     spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
     spring.datasource.driver-class-name=com.mysql.jdbc.Driver
     spring.datasource.url=jdbc:mysql://localhost:3306/druid
     spring.datasource.username=root
     spring.datasource.password=root
     
     # 下面为连接池的补充设置，应用到上面所有数据源中
     # 初始化大小，最小，最大
     spring.datasource.druid.initialSize=5
     spring.datasource.druid.minIdle=5
     spring.datasource.druid.maxActive=20
     # 配置获取连接等待超时的时间
     spring.datasource.druid.maxWait=60000
     # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
     spring.datasource.druid.timeBetweenEvictionRunsMillis=60000
     # 配置一个连接在池中最小生存的时间，单位是毫秒
     spring.datasource.druid.minEvictableIdleTimeMillis=300000
     # 测试连接是否有效的sql
     spring.datasource.druid.validationQuery=SELECT 1 FROM DUAL
     #spring.datasource.validationQuery=select 'x'
     # 建议配置为true，不影响性能，并且保证安全性
     # 申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效
     spring.datasource.druid.testWhileIdle=true
     # 申请连接时执行validationQuery检测连接是否有效
     spring.datasource.druid.testOnBorrow=false
     # 归还连接时执行validationQuery检测连接是否有效
     spring.datasource.druid.testOnReturn=false
     # 打开PSCache，并且指定每个连接上PSCache的大小
     spring.datasource.druid.poolPreparedStatements=true
     spring.datasource.druid.maxPoolPreparedStatementPerConnectionSize=20
     # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
     # 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：
     # 监控统计用的filter:stat
     # 日志用的filter:log4j
     # 防御sql注入的filter:wall
     spring.datasource.druid.filters=stat,wall,log4j
     # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
     spring.datasource.druid.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
     # 合并多个DruidDataSource的监控数据
     #spring.datasource.useGlobalDataSourceStat=true
     ```

     在properties文件中，写上spring.datasouce.initialsize时就显示了druid，所以最后的属性不一定对，idea会显示的。

   + 第二种方式（配置druid的监控功能用的，普通配置用第一种方式就行）

     Druid配置数据库连接池，mybatis操作数据库的核心是SqlSessionTemplate，除此之外还有几个属性，比如SqlSessionFactory、DataSource。DataSource用来抽象数据库对象，SqlSessionFactory会话工厂，用来产生SQLSession。SqlSessionTemplate是mybatis的Mapper类中的重要属性，mapper方法通过该属性进行数据库的操作。具体代码如下：
   
     ```java
     @Configuration
     @MapperScan(basePackages = "com.kevin.study.springboot.domain.mapper.mysql",sqlSessionFactoryRef = "mysqlSessionFactory")
     public class MysqlDataSourceConfig {
     
         @Bean
         public DataSource getMysqlDataSource(){
             DruidDataSource ds = new DruidDataSource();
             ds.setUrl("jdbc:mysql://ipaddress:3306/agtm?useUnicode=true&characterEncoding=UTF-8");
             ds.setUsername("db");
             ds.setPassword("123456");
             ds.setMaxActive(100);
             ds.setTestOnBorrow(false);
             return ds;
         }
         @Bean(name = "mysqlSessionFactory")
         public SqlSessionFactory getMysqlSessionFactory() throws Exception {
             SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
             factory.setDataSource(this.getMysqlDataSource());
             factory.setVfs(SpringBootVFS.class);
             return factory.getObject();
         }
         @Bean(name = "mysqlSessionTemplate")
         public SqlSessionTemplate getSqlSessionTemplate() throws Exception {
             return  new SqlSessionTemplate(this.getMysqlSessionFactory());
         }
  }
     ```

     **注意：**

     (1)、类必须使用@Configuration注解，只有这样springboot才可以将该类中@Bean方法作为托管类对象，托管到IOC上下文中，不使用注解的话，会报如下错误：Cannot determine embedded database driver class for database type NONE 和 No bean named 'mysqlSessionFactory' is defined。其中第一个是因为不使用Configuration注解类，导致找不到SqlSessionFactory导致的，第二个是不使用Bean注解方法，导致MapperScan中的sqlSessionFactoryRef 指向，没有托管的对象。

     (2)、DataSource 使用的是DruidDataSource，该对象还有很多属性，具体可百度，不赘述

     (3)、SqlSessionFactory是使用的SqlSessionFactoryBean来实例化的

     (4)、三者的具体层次关系为：

     SqlSessionTemplate

     属性：SqlSessionFactory
   
     属性：DataSource
   
3. 配置监控功能

   在config包中建如下java类，并写上如下代码

   ```java
   @Configuration
   public class DruidConfiguration {
   
       @Bean
       public ServletRegistrationBean statViewServle(){
           ServletRegistrationBean servletRegistrationBean=new ServletRegistrationBean(new StatViewServlet(),"/druid/*");
           //IP白名单
           //servletRegistrationBean.addInitParameter("allow","192.168.1.12,127.0.0.1");
           //IP黑名单
           //servletRegistrationBean.addInitParameter("deny","192.168.4.23");
           //控制台用户
           servletRegistrationBean.addInitParameter("loginUsername","admin");
           servletRegistrationBean.addInitParameter("loginPassword","123456");
           //是否能够重置数据
           servletRegistrationBean.addInitParameter("resetEnable","false");
           return servletRegistrationBean;
       }
       @Bean
       public FilterRegistrationBean statFilter(){
           FilterRegistrationBean filterRegistrationBean=new FilterRegistrationBean(new WebStatFilter());
           //添加过滤规则
           filterRegistrationBean.addInitParameter("exclusions","*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
           return filterRegistrationBean;
       }
   }
   ```

   访问监控 http://localhost/druid/weburi.html ，输入java代码里配置的用户名和密码即可

