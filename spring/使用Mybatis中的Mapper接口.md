## 使用Mybatis中的Mapper接口

### 一、Mapper中的方法介绍

举例如下：

|                             方法                             |                           功能说明                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| int countByExample(UserExample example) thorws SQLException  |                          按条件计数                          |
|    int deleteByPrimaryKey(Integer id) thorws SQLException    |                          按主键删除                          |
| int deleteByExample(UserExample example) thorws SQLException |                          按条件查询                          |
|    String/Integer insert(User record) thorws SQLException    |                    插入数据（返回值为ID）                    |
|   User selectByPrimaryKey(Integer id) thorws SQLException    |                          按主键查询                          |
| List selectByExample(UserExample example) thorws SQLException |                          按条件查询                          |
| List selectByExampleWithBLOGs(UserExample example) throws SQLException | 按条件查询（包括BLOB字段）。只有当数据表中的字段类型有为二进制的才会产生。 |
| int updateByPrimaryKeySelective(User record) thorws SQLException |                  按主键更新值不为null的字段                  |
|   int updateByPrimaryKey(User record) thorws SQLException    |                          按主键更新                          |
| int updateByExample(User record, UserExample example) thorws SQLException |                          按条件更新                          |
| int updateByExampleSelective(User record, UserExample example) thorws SQLException |                  按条件更新值不为null的字段                  |

小结：

+ 凡是按条件查询的，都需要用对应实例的example，相当于sql语句中where后面的部分
+ 带有Selective都是操作不为null的字段



### 二、example用法

example用于添加条件，用法如下：

```java
xxxExample example = new xxxExample();
Criteria criteria = new Example().createCriteria();
```

|                    方法                    |                     说明                      |
| :----------------------------------------: | :-------------------------------------------: |
|  example.setOrderByClause(“字段名 ASC”);   |         添加升序排列条件，DESC为降序          |
|         example.setDistinct(false)         | 去除重复，boolean型，true为选择不重复的记录。 |
|           criteria.andXxxIsNull            |            添加字段xxx为null的条件            |
|          criteria.andXxxIsNotNull          |           添加字段xxx不为null的条件           |
|       criteria.andXxxEqualTo(value)        |           添加xxx字段等于value条件            |
|      criteria.andXxxNotEqualTo(value)      |          添加xxx字段不等于value条件           |
|     criteria.andXxxGreaterThan(value)      |           添加xxx字段大于value条件            |
| criteria.andXxxGreaterThanOrEqualTo(value) |         添加xxx字段大于等于value条件          |
|       criteria.andXxxLessThan(value)       |           添加xxx字段小于value条件            |
|  criteria.andXxxLessThanOrEqualTo(value)   |         添加xxx字段小于等于value条件          |
|        criteria.andXxxIn(List<？>)         |          添加xxx字段值在List<？>条件          |
|       criteria.andXxxNotIn(List<？>)       |         添加xxx字段值不在List<？>条件         |
|     criteria.andXxxLike(“%”+value+”%”)     |      添加xxx字段值为value的模糊查询条件       |
|   criteria.andXxxNotLike(“%”+value+”%”)    |     添加xxx字段值不为value的模糊查询条件      |
|   criteria.andXxxBetween(value1,value2)    |     添加xxx字段值在value1和value2之间条件     |
|  criteria.andXxxNotBetween(value1,value2)  |    添加xxx字段值不在value1和value2之间条件    |



### 三、应用举例

#### 1. 查询数据

+ selectByPrimaryKey()

  ```java
  User user = XxxMapper.selectByPrimaryKey(100); //相当于select * from user where id = 100
  ```

+ selectByExample() 和 selectByExampleWithBLOGs()

  ```java
  UserExample example = new UserExample();
  Criteria criteria = example.createCriteria();
  criteria.andUsernameEqualTo("wyw");
  criteria.andUsernameIsNull();
  example.setOrderByClause("username asc,email desc");
  List<?>list = XxxMapper.selectByExample(example);
  //相当于：select * from user where username = 'wyw' and  username is null order by username asc,email desc
  ```

+ 注：在逆向工程生成的文件XxxExample.java中包含一个static的内部类Criteria，Criteria中的方法是定义SQL 语句where后的查询条件。

#### 2. 插入数据

+ insert()

  ```java
  User user = new User();
  user.setId("dsfgsdfgdsfgds");
  user.setUsername("admin");
  user.setPassword("admin")
  user.setEmail("wyw@163.com");
  XxxMapper.insert(user);
  //相当于：insert into user(ID,username,password,email) values ('dsfgsdfgdsfgds','admin','admin','wyw@126.com');
  ```

#### 3. 更新数据

+ updateByPrimaryKey()

  ```java
  User user =new User();
  user.setId("dsfgsdfgdsfgds");
  user.setUsername("wyw");
  user.setPassword("wyw");
  user.setEmail("wyw@163.com");
  XxxMapper.updateByPrimaryKey(user);
  //相当于：update user set username='wyw', password='wyw', email='wyw@163.com' where id='dsfgsdfgdsfgds'
  ```

+ updateByPrimaryKeySelective()

  ```java
  User user = new User();
  user.setId("dsfgsdfgdsfgds");
  user.setPassword("wyw");
  XxxMapper.updateByPrimaryKeySelective(user);
  //相当于：update user set password='wyw' where id='dsfgsdfgdsfgds'
  ```

+ updateByExample() 和 updateByExampleSelective()

  ```java
  UserExample example = new UserExample();
  Criteria criteria = example.createCriteria();
  criteria.andUsernameEqualTo("admin");
  User user = new User();
  user.setPassword("wyw");
  XxxMapper.updateByPrimaryKeySelective(user,example);
  //相当于：update user set password='wyw' where username='admin'
  ```

  updateByExample()更新所有的字段，包括字段为null的也更新，建议使用 updateByExampleSelective()更新想更新的字段

#### 4. 删除数据

+ deleteByPrimaryKey()

  ```java
  XxxMapper.deleteByPrimaryKey(1);  //相当于：delete from user where id=1
  ```

+ deleteByExample()

  ```java
  UserExample example = new UserExample();
  Criteria criteria = example.createCriteria();
  criteria.andUsernameEqualTo("admin");
  XxxMapper.deleteByExample(example);
  //相当于：delete from user where username='admin'
  ```

#### 5. 查询数据数量

+ countByExample()

  ```java
  UserExample example = new UserExample();
  Criteria criteria = example.createCriteria();
  criteria.andUsernameEqualTo("wyw");
  int count = XxxMapper.countByExample(example);
  //相当于：select count(*) from user where username='wyw'
  ```

  

