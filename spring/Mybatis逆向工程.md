## Mybatis逆向工程

### 1、修改pom.xml

在pom.xml中的plugin部分添加逆向工程的maven插件

```xml
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.4.0</version>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>8.0.12</version>
                        <scope>runtime</scope>
                    </dependency>
                    <dependency>
						<groupId>org.mybatis.generator</groupId>
						<artifactId>mybatis-generator-core</artifactId>
						<version>1.4.0</version>
					</dependency>
                </dependencies>
                <executions>
                    <execution>
                        <id>Generate MyBatis Artifacts</id>
                        <phase>package</phase>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <!--允许移动生成的文件 -->
                    <verbose>true</verbose>
                    <!-- 是否覆盖 -->
                    <overwrite>true</overwrite>
                    <!-- 自动生成的配置 -->
                    <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
                </configuration>
            </plugin>
```

### 2、配置映射文件

在`<configurationFile>`配置的对应目录下，创建对应的逆向生成工程映射文件，并将下列配置粘贴到其中：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC
        "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <context id="DB2Tables" targetRuntime="MyBatis3">

        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true" />
        </commentGenerator>
		
        <!--数据库连接的信息：驱动类、连接地址、用户名、密码 ,可选加上“useSSL=false” -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/ssp?serverTimezone=GMT%2B8"
                        userId="root"
                        password="13610214">
            <property name="nullCatalogMeansCurrent" value="true"/>
        </jdbcConnection>
		
        <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 
			和 NUMERIC 类型解析为java.math.BigDecimal -->
        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
		
        <!-- targetProject:生成pojo类的位置 -->
        <javaModelGenerator targetPackage="edu.qingtai.ssp.domain" targetProject=".\src\main\java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
            <!-- 从数据库返回的值被清理前后的空格 -->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
		
        <!-- targetProject:mapper映射文件生成的位置(xml) -->
        <sqlMapGenerator targetPackage="resources.mybatis"  targetProject=".\src\main">
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>
		
        <!-- targetPackage：mapper接口生成的位置 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="edu.qingtai.ssp.mapper"  targetProject=".\src\main\java">
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>
		
        <!-- 指定数据库表tableName -->
        <table schema="" tableName="seminar"> </table>
        <table schema="" tableName="..."> </table>

    </context>
</generatorConfiguration>
```

将 mapper 映射的 xml 文件注册到 springboot：在`application.properties`配置文件中增加映射文件扫描：

```properties
mybatis.mapper-locations=classpath:mybatis/*.xml #classpath指的是resources目录
```

并且要在mapper接口上增加`@Mapper`注解，或者使用包扫描的方式进行注册：在启动类上增加`@MapperScan(basePackages="org.woodwhale.demo.mapper")`

### 3、运行逆向工程

+ 使用Java代码的方式

  在Utils包中增加类MyBatisGeneratorApp.java

  ```java
  public class MyBatisGeneratorApp {
  
  	public void generator() throws Exception{
  
  		List<String> warnings = new ArrayList<String>();
  		boolean overwrite = true;
  		//指定 逆向工程配置文件
  		File configFile = new File("generatorConfig.xml"); 
  		ConfigurationParser cp = new ConfigurationParser(warnings);
  		Configuration config = cp.parseConfiguration(configFile);
  		DefaultShellCallback callback = new DefaultShellCallback(overwrite);
  		MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
  				callback, warnings);
  		myBatisGenerator.generate(null);
  
  	} 
  	public static void main(String[] args) throws Exception {
  		try {
  			MyBatisGeneratorApp generatorSqlmap = new MyBatisGeneratorApp();
  			generatorSqlmap.generator();
  		} catch (Exception e) {
  			e.printStackTrace();
  		}
  		
  	}
  
  }
  ```

+ 使用maven插件的方式

  + 1、在项目根目录右击选择`"Show in"`中的`"Terminal"`选项，进入 dos 窗口，执行逆向生成命令：`mvn mybatis-generator:generate`，项目生成之后，需要刷新项目。

  + 2、双击图片中的generate

    ![image-20200224002307147](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200224002307147.png)

### 4、更多配置文件xml的参数详见[链接](https://woodwhales.cn/2018/10/29/015/)