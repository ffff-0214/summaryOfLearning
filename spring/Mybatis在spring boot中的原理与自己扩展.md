## Mybatis在spring boot中的原理与自己扩展

### 一、原理

1. 数据层domain

   将数据库的表的属性，变成domain包中的实体

2. 接口层mapper

   将mapper定义为接口，只定义***方法***，所以是interface类

3. xml文件

   是mapper层接口的具体实现，与mapper文件通过namespace 形成映射

4. service服务

   使用mapper中的接口，实现自己的服务

### 二、实战

已用逆向工程生成，但是需要添加分页的查询操作，此markdown只讲流程

需求：根据List批量查询List结果

1. 在mapper文件中添加需要用于查询数据库的接口

   ![image-20200304144024101](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200304144024101.png)

2. 在xml文件中写上具体实现

   collection是传参名称，item是list里面的实体参数名称

   ![image-20200304144125920](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200304144125920.png)

   open,separator,close经常用于自定义实体类时，举个例子：

   ```xml
   <insert id="addEmpsByList" parameterType="com.jas.mybatis.bean.Employee">
        INSERT INTO t_employee(username, gender, email) VALUES 
        <foreach collection="list" item="emp" separator=",">
            (#{emp.username}, #{emp.gender}, #{emp.email})
        </foreach>
   </insert>
   ```

   注意sql语句SELECT * FROM table WHERE id IN () 这个会报错，所以最后判断一下查询的列表中是有元素的

3. 在service中写上对应的服务接口，在serviceImpl中进行具体的实现，需要时调用mapper的接口即可