### 安装MySQL

#### 1、卸载自带的mariadb，安装依赖环境

+ 列出已安装的mariadb并卸载

  > ```shell
  > [root@mysql ~]# rpm -qa | grep mariadb
  > mariadb-libs-5.5.60-1.el7_5.x86_64
  > 方法一：rpm -e --nodeps 卸载 mariadb
  > [root@mysql ~]# rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
  > 方法二：yum -y remove 卸载 mariadb
  > [root@mysql ~]# yum -y remove mariadb-libs-5.5.60-1.el7_5.x86_64
  > ```

+ 删除多余的mysql残留项目

  > ```shell
  > [root@mysql ~]# find / -name mysql -print
  > 如下：
  > [root@mysql ~]# rm -rf /var/lib/mysql
  > [root@mysql ~]# rm -rf /var/lib/mysql/mysql
  > [root@mysql ~]# rm -rf /usr/bin/mysql
  > [root@mysql ~]# rm -rf /usr/lib64/mysql
  > ```

+ 安装基础和依赖软件（可以先不装，最后缺啥再装）

  > ```shell
  > [root@mysql ~]# yum -y install wget (如果你不用ftp或者lrzsz上传)
  > [root@mysql ~]# yum -y install lrzsz (简单好用的上传下载软件)
  > [root@mysql ~]# yum -y install vim
  > 
  > [root@mysql ~]# yum -y install net-tools
  > [root@mysql ~]# yum -y install openssl openssl-devel
  > [root@mysql ~]# yum -y install libaio libaio-devel
  > [root@mysql ~]# yum -y install perl perl-devel
  > [root@mysql ~]# yum -y install perl-JSON.noarch
  > [root@mysql ~]# yum -y install autoconf
  > ```

+ 关闭selinux

  > ```shell
  > [root@mysql home]# vi /etc/selinux/config #将SELINUX=enforcing 改为SELINUX=disabled
  > ```

+ 重启CentOS

  ```shell
  [root@mysql home]# reboot
  ```

#### 2、上传tar包并安装

+ 上传mysql的tar包，解压tar包

  ``` shell
  [root@mysql home]# tar -xvf mysql-8.0.17-1.el7.x86_64.rpm-bundle.tar
  ```

  如果服务器没有装zmodem传输，则需要装：

  ```bash
  # yum install lrzsz
  ```

  太慢！还是用ftp好

+ 开始安装

  > ```shell
  > PS：注意安装顺序 common -> libs -> client -> server
  > [root@mysql home]# rpm -ivh mysql-community-common-8.0.17-1.el7.x86_64.rpm
  > [root@mysql home]# rpm -ivh mysql-community-libs-8.0.17-1.el7.x86_64.rpm
  > [root@mysql home]# rpm -ivh mysql-community-client-8.0.17-1.el7.x86_64.rpm
  > [root@mysql home]# rpm -ivh mysql-community-server-8.0.17-1.el7.x86_64.rpm
  > 
  > [root@mysql home]# rpm -ivh mysql-community-libs-compat-8.0.17-1.el7.x86_64.rpm（可选）
  > [root@mysql home]# rpm -ivh mysql-community-devel-8.0.17-1.el7.x86_64.rpm（可选）
  > [root@mysql home]# rpm -ivh mysql-community-embedded-compat-8.0.17-1.el7.x86_64.rpm（可选）
  > [root@mysql home]# rpm -ivh mysql-community-test-8.0.17-1.el7.x86_64.rpm（可选）
  > ```
  >
  > *红帽*安装rpm安装MySQL时爆出警告： 警告：MySQL-server-5.5.46-1.linux2.6.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY 原因：这是由于yum安装了旧版本的GPG keys造成的，解决办法：
  >
  > + 后面加上  --force --nodeps 如：  rpm -ivh MySQL-server-5.5.46-1.linux2.6.x86_64.rpm --force --nodeps 从 RPM 版本 4.1 开始，在安装或升级软件包时会检查软件包的签名
  > + 引用 
  >   rpm --import /etc/pki/rpm-gpg/RPM* 

+ 初始化数据库，目录授权，并启动

  > ```shell
  > [root@mysql home]# mysqld --initialize --console
  > [root@mysql home]# chown -R mysql:mysql /var/lib/mysql/
  > [root@mysql home]# systemctl start mysqld
  > [root@mysql home]# systemctl status mysqld #查看是否启动成功
  > ```





---



### 删除MySQL

#### 1、查看mysql安装了哪些东西



```undefined
rpm -qa |grep -i mysql
```

![img](https:////upload-images.jianshu.io/upload_images/7802645-bfcd164758838157.png?imageMogr2/auto-orient/strip|imageView2/2/w/584/format/webp)



#### 2、开始卸载



```shell
yum remove mysql-community-common-5.7.20-1.el7.x86_64
yum remove mysql-community-client-5.7.20-1.el7.x86_64
yum remove mysql57-community-release-el7-11.noarch #rpm源可以不用删，留着让yum再次下载
yum remove mysql-community-libs-5.7.20-1.el7.x86_64
yum remove mysql-community-server-5.7.20-1.el7.x86_64
```

#### 3、查看是否卸载完成

![img](https:////upload-images.jianshu.io/upload_images/7802645-ed6095d36bf593b6.png?imageMogr2/auto-orient/strip|imageView2/2/w/372/format/webp)



#### 4、查找mysql相关目录



```swift
find / -name mysql
```

![img](https:////upload-images.jianshu.io/upload_images/7802645-052d769f66b90b6f.png?imageMogr2/auto-orient/strip|imageView2/2/w/394/format/webp)

#### 5、删除相关目录



```undefined
rm -rf 
```

![img](https:////upload-images.jianshu.io/upload_images/7802645-f334e2d6474898a9.png?imageMogr2/auto-orient/strip|imageView2/2/w/412/format/webp)



#### 6、删除/etc/my.cnf



```undefined
rm -rf /etc/my.cnf
```

#### 7、删除/var/log/mysqld.log

（如果不删除这个文件，会导致新安装的mysql无法生存新密码，导致无法登陆）

```cpp
rm -rf /var/log/mysqld.log
```

