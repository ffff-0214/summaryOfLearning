## Mybatis中的遇到的一些error

### Could not autowire.No beans of 'xxxMapper' type found.

此错误发生在声明mapper但不实例化时发生。程序是可以运行起来的，是因为`@mapper`接口并不能在Spring boot中自动注册bean

解决方法：

+ 在mapper文件中用`@Repository`注解

  ![image-20200304143107909](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200304143107909.png)

+ 在启动类中修改`@MapperScan`

  ![image-20200304143247268](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200304143247268.png)

### org.apache.ibatis.binding.BindingException: Parameter '***' not found

+ 在mapper接口映射xml文件传参数时，xml文件用#{}接收，报出此错误

+ 在mapper接口中传参数时加上`@Param("parameterName")`传参

  ![image-20200304145108782](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200304145108782.png)

### java.lang.UnsupportedOperationException

此错误发生在用mybatis映射SQL语句的时候出现异常，问题所在resultType="返回结果类型"

resultType的使用：

+ 基本类型 ：resultType=基本类型

  如果返回值为基本类型，则resultType=基本类型，比如resultType=java.lang.Integer,那sql语句中只返回一个int类型数据，通常用于统计数量

+ List类型： resultType=List中元素的类型

  如果返回值为list类型，则resultType=List中元素的类型，比如你需要返回一个List类型的数据，那么这里resultType=”java.lang.String”，如果需要返回一个实体类，那么resultType=”com.pjf.mybatis.car”以此类推

+ Map类型 

  + 单条记录：resultType =map

    如果返回值为map单条类型，比如{username=”张三”}，那么resultType =”map”

  + 多条记录：resultType =Map中value的类型
    如果返回值为map多条记录，比如{res=”实体类”}，实体类就是你要请求的数据实体类，那么resultType =Map中value的类型，比如resultType=”com.pjf.mybatis.car”

### org.apache.ibatis.type.TypeException: Could not resolve type alias 'Student'

+ 第一种办法：用自己定义的实体类时一定要把***包名写全***即可

+ 第二种办法：配置全局xml

  + 在resource目录下创建一个mybatis-config.xml文件，文件内容为参考[链接1](https://www.jianshu.com/p/2756e81d02ff)，[链接2](https://blog.csdn.net/majinggogogo/article/details/71503263?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)，主要是配置别名

    ```xml
    <typeAliases >
        <typeAlias type="cn.huan.mybatis.AdminBean" alias="AdminBean"/></typeAlias>
    </typeAliases>
    ```

  + 在application.properties中加入mybatis的配置

    ```properties
    #mybatis的mapper配置文件
    mybatis.config-location=classpath:mybatis-config.xml  # mybatis配置文件所在路径
    mybatis.mapper-locations=classpath:.../*.xml   # 所有的mapper映射文件
    mybatis.type-aliases-package=com.becl.dao.mapper # 定义所有操作类的别名所在包
    ```

    此方法没有问题

### mybatis取出mysql中Date类型时出现日期格式不对应的情况

错误定位：

+ mysql中的date类型存储"2020-01-08"
+ 在java后台print出"Fri Feb 26 16:33:08 CST 2016"字符串类型
+ 在controller传给前端时显示"2016-08-15T16:00:00.000Z"的时间格式
+ 根本原因是时区和通用标准不一样

可以通过SimpleDateFormat进行转换，但是如果List很大，效率很低

解决方案：

+ 在数据库层实体类的定义中，将java.util.date转换成java.sql.date

