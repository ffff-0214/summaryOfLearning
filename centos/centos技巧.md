## yum安装过程中止

> ctrl+z  #中断当前的安装显示
>
> ps -ef | grep yum  #查找当前yum相关的进程
>
> kill -9 进程号(pid)  #杀掉进程

## yum下载过慢

换源

> 1、备份
> mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
> 2、下载源
>
> wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
> 或者
> wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
>
> yum clean all
>

可能会出现：wget: unable to resolve host address

> 解决办法：
>
> 进入/etc/resolv.conf。
>
> 修改内容为下
> nameserver 8.8.8.8 #google域名服务器
> nameserver 8.8.4.4 #google域名服务器