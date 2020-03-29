## 使用CentOS云服务器

### 一、查看CentOS版本

+ cat /etc/redhat-release #版本
+ uname -r #内核

### 二、查看安装了什么软件

+ rpm -qa | grep ...
+ rpm -qla|grep ...   #查看安装位置
+ rpm -qa | sort #查看所有安装了的软件包

#### yum和rpm的区别与联系

1. rpm :RedHat package manage的简写
   rpm 是linux的一种软件包名称，以.rmp结尾，安装的时候语法为：rpm -ivh，rpm包的安装有一个很大的缺点就是文件的关联性太大，有时候装一个软件要安装很多其他的软件包，很麻烦，
2. yum（全称为 Yellow dog Updater, Modified）
   yum是一个在Fedora和RedHat以及SUSE中的Shell前端软件包管理器。基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。yum提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。
   为了解决rpm安装时文件关联行太大的问题,RedHat小红帽开发了yum安装方法，他可以彻底解决这个关联性的问题，很方便，只要配置两个文件即可安装，安装方法是：yum -y install ，yum并不是一中包，而是安装包的软件
   yum的命令形式一般是如下：yum [options] [command] [package ...]
   其中的[options]是可选的，选项包括-h（帮助），-y（当安装过程提示选择全部为"yes"），-q（不显示安装的过程）等等。[command]为所要进行的操作，[package ...]是操作的对象。
3. 区别
   rpm 只能安装已经下载到本地机器上的rpm 包. yum能在线下载并安装rpm包,能更新系统,且还能自动处理包与包之间的依赖问题,这个是rpm 工具所不具备的。

### 三、查看可以登录此服务器上的用户

+ cd  /home再ls，如果有，就是用户名，如果没有就是没有可以登录的用户

### 四、安装软件到哪

Linux 的软件安装目录是也是有讲究的，理解这一点，在对系统管理是有益的

`/usr`：系统级的目录，可以理解为`C:/Windows/`，`/usr/lib`理解为`C:/Windows/System32`。
`/usr/local`：用户级的程序目录，可以理解为`C:/Progrem Files/`。用户自己编译的软件默认会安装到这个目录下，安装时要记得新建软件目录
`/opt`：用户级的程序目录，可以理解为`D:/Software`，opt有可选的意思，这里可以用于放置第三方大型软件（或游戏），当你不需要时，直接`rm -rf`掉即可。在硬盘容量不够时，也可将/opt单独挂载到其他磁盘上使用。

源码放哪里？
`/usr/src`：系统级的源码目录
`/usr/local/src`：用户级的源码目录



以prefix进行安装

  --prefix 将软件包安装到由 指定的路径下，例如到/opt下

  rpm -ivh --prefix= /opt xxx.rpm

### 五、端口问题

如果服务器中的端口开放了，但是还需要去[阿里云](https://blog.csdn.net/kaifaxiaoliu/article/details/80403736?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)的实例安全组规则看看