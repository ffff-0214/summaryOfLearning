### 在centos上初始启动mysql

---

#### 一、执行Mysql服务命令

+ 启动命令

```shell
systemctl start mysqld
```

+ 查看状态

```shel
systemctl status mysqld
```

+ 关闭命令

```shel
 systemctl stop mysqld
```

+ 重启命令

```shel
systemctl restart mysqld
```

---

#### 二、修改密码

+ ##### 初始启动以后，Mysql默认创建了root用户的密码，输出在MySQL 的日志文件`/var/log/mysqld.log`中，可以通过`temporary password`关键字来找出这个临时的密码

```shell
grep 'temporary password' /var/log/mysqld.log
```

+ ##### 找到密码之后使用改密码连接数据库：

```shell
mysql -u root -p
```

​		然后修改密码

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPassword';
```

​		执行上述命令密码将被修改为：` NewPassword`

​		PS：新版本的MySQL对密码强度有限制，执行到上一步的时候，会	提示密码强度不够，则应更改为更高强度的密码。**（不到万不得已不要改！）**

```shell
#查看密码策略
show variables like '%validate_password.policy%';
show variables like '%validate_password.length%';
#修改密码策略
set global validate_password.policy=0;  #设置为弱口令
set global validate_password.length=1;  #密码最小长度为1
```

> validate_password.policy（校验规则），取值范围[0,1,2]，默认值1。0（LOW）：只校验长度；1（MEDIUM）：校验长度、大小写和特殊字符；2（STRONG）：校验长度、大小写、特殊字符和dictionary_file

+ ##### 密码更改完成之后重启MySQL服务：

```
systemctl restart mysqld.service
```

---

#### 三、修改mysql默认端口

+ ##### 编辑配置文件，将[mysqld]下面的port = 3306改成port = 3307

  没有的需要添加`port=<yourport>`

  ```shell
  vim /etc/my.cnf
  ```

  重启数据库

  ```shell
  systemctl restart mysqld.service
  ```

  改端口后，重启可能会失败，提示的解决方案都不管用，用命令查看mysql日志找[ERROR]

  ```shell
  #查看日志
  more /var/log/mysqld.log
  ```

  再去谷歌上查是什么错误，这里解决两种错误：

  1. 权限问题，缺少目录文件mysqld.pid，给mysql权限去创建

     ``` shell
     chown  mysql.mysql /var/run/mysqld/
     chown -R mysql:mysql /var/lib/mysql/
     ```

  2. selinux服务，此服务拒绝tcp/ip连接，保护centos中的权限分配

     ```shell
     #关掉 selinux，不重启centos
     /usr/sbin/setenforce 0
     #需要重启centos，需要修改： 
     vim /etc/selinux/config #使得SELINUX=disabled
     ```

+ ##### 开放防火墙端口

  > 1. 开启端口
  >
  > ```shell
  > firewall-cmd --zone=public --add-port=3307/tcp --permanent
  > ```
  >
  > 命令含义：
  >
  > --zone #作用域
  >
  > --add-port=80/tcp  #添加端口，格式为：端口/通讯协议
  >
  > --permanent  #永久生效，没有此参数重启后失效
  >
  > 2. 开启端口后需要重启防火墙
  >
  > ```shell
  > systemctl restart firewalld
  > ```
  >
  > 3. 查看已经开放的端口，这时就可以看到3307/tcp
  >
  > ```shell
  > firewall-cmd --list-ports
  > ```

---

#### 四、设置与navicat的连接

+ ##### 问题1：新建的mysql默认不允许远程访问，有[两种解决办法](https://www.cnblogs.com/liangzhihong/p/10452207.html)

  1. 改为所有可远程的连接

     >MySQL默认只对本机开放连接，我们则需要对mysql表的host字段进行修改以支持其他主机连接,%表示所有。
     >
     >```mysql
     ># 先连接数据库
     >use mysql;
     >update user set host = '%' where user = 'root';
     >```
     >
     >更改完成之后刷新权限：
     >
     >```mysql
     >flush privileges;
     >```

  2. 指定主机ip指定密码连接此数据库

     > root使用mypassword从任何主机连接到mysql服务器
     >
     > ```mysql
     > GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
     > ```
     >
     > 允许用户myuser从ip为192.168.1.3的主机连接到mysql服务器，并使用mypassword作为密码 
     >
     > ```mysql
     > GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.1.3' IDENTIFIED BY 
     > 　　'mypassword' WITH GRANT OPTION; 
     > ```
     >
     > 更改完成之后刷新权限：
     >
     > ```mysql
     > flush privileges;
     > ```

+ ##### 问题2：出现2059错误**(Navicat12会出现此错误，Navicat15不会出现)**

  `Authentication plugin 'caching_sha2_password' cannot be loaded`

  mysql8.0 引入了新特性 caching_sha2_password；这种密码加密方式客户端不支持；客户端支持的是mysql_native_password 这种加密方式；

  1. 两种修改mysql密码加密方式的方法

     > + 修改配置文件
     >
     >   default_authentication_plugin=caching_sha2_password 修改为：
     >
     >   default_authentication_plugin=mysql_native_password
     >
     > + 使用命令进行更改
     >
     >   alter USER 'root'@'localhost' identified BY 'password' PASSWORD EXPIRE NEVER; #修改加密规则
     >   alter USER 'root'@'localhost' identified WITH mysql_native_password BY 'password'; #更新一下用户的密码
     >
     >   alter user 'root'@'localhost' identified by '你自己的密码'; #再重置下密码：
     >   flush privileges; #刷新权限

  2. ~~修改库mysql的表user即可~~**（千万不要用这种方法）**

     > 我们可可以查看mysql 数据库中user表的 plugin字段；
     >
     > ![img](https://img-blog.csdn.net/20180422181257277)
     >
     > 可以使用命令将他修改成mysql_native_password加密模式：
     >
     > ```mysql
     > update user set plugin='mysql_native_password' where user='root';
     > ```