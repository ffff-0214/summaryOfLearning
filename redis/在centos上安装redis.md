## 在centos上安装redis

### 一、将tar文件放到home目录下

将安装包放置到/opt下，不要像我这样

![image-20200311121751589](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200311121751589.png)

### 二、解压

```
tar -zxvf redis-5.0.5.tar.gz

```

解压到指定目录：一般在 /usr/local中是用户级软件，所以要解压到那里去

```shell
# tar -zxvf /bbs.tar.zip -C /zzz/bbs 
```

+ 把根目录下的bbs.tar.zip解压到/zzz/bbs下，前提要保证存在/zzz/bbs这个目录 
+ 这个和cp命令有点不同，cp命令如果不存在这个目录就会自动创建这个目录！
+ 附：用tar命令打包，将当前目录下的zzz文件打包到根目录下并命名为zzz.tar.gz
  　　#tar zcvf /zzz.tar.gz ./zzz

### 三、编译

注意要进入到redis文件夹中

![image-20200311140225984](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200311140225984.png)

+ 报错1

  ![image-20200311140711696](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200311140711696.png)

  解决方案：安装gcc

  ``` 
  yum install gcc
  ```

+ 报错2

  ![image-20200311141026762](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200311141026762.png)

  解决方案：原因是jemalloc重载了Linux下的ANSI C的malloc和free函数。解决办法：make时添加参数。

  ``` 
  make MALLOC=libc
  ```

+ 编译好 Redis 之后，可以使用 `make test` 命令测试一下

  可能出现提示 `You need tcl 8.5 or newer in order to run the Redis test` ，这是缺少 tcl 包，安装一下 tcl 就好了（如 `yum install tcl`）

### 四、安装

测试完成，就可以安装 Redis 了，先 cd 到 Redis 解压文件的 src 目录，使用 `make PREFIX=/usr/local/redis install` 安装，可以设置 Redis 的安装位置

下面有说明为什么：

``` 
make PREFIX=/usr/local/redis install
```

### 五、配置

看懂原理即可，有点乱。

安装完 Redis，可以看到 Redis 安装目录下只有一个 bin 目录，目录内容如下：

- redis-server —— Redis 的服务器
- redis-cli —— Redis 的命令行客户端
- redis-benchmark —— Redis 的性能测试工具
- redis-check-rdb —— Redis 的 RDB 文件检索工具
- redis-check-aof —— Redis 的 AOF 文件修复工具
- redis-sentinel —— Redis 的集群监控工具

将编译后的目录下的redis.conf拷贝到/etc文件夹的新建/redis文件夹下

![image-20200311143029475](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200311143029475.png)

编辑/etc/redis下的配置文件

```csharp
mkdir /usr/local/redis/logs
mkdir /usr/local/redis/datas
```

新建两个文件夹后，配置：

```bash
# 绑定地址去除回环地址，添加物理地址
#绑定的IP地址，默认只允许本机访问 Redis，也即是 127.0.0.1（localhost），如果其他IP也想访问，可以将 bind 127.0.0.1 改为 bind 指定的IP地址，IP 地址设置成 0.0.0.0 表示允许任何IP访问，但这样做不安全
#bind 127.0.0.1
bind 172.16.1.20
# 指定以守护进程方式启动
daemonize yes
# 指定日志文件夹
logfile "/usr/local/redis/logs/redis.log"
# 指定数据文件夹
dir /usr/local/redis/datas
```

### 六、启动

指定配置文件启动

``` 
redis-server  /usr/local/etc/redis.conf 
```

+ 报错

  bash: redis-server: command not found...

  + 解决方案1

    之前使用命令

    ``` 
    make PREFIX=/usr/local/redis install
    ```

    是将redis的命令放到了/usr/local/redis中，如果要使用redis-server则需要写完整

    这样的好处是使得redis在可控的文件夹下，不至于很乱

  + 解决方案2

    在原始redis-5.0.5的文件夹下使用命令

    ``` 
    make install
    ```

    实际上，就是将这个几个文件加到/usr/local/bin目录下去。这个目录在Path下面的话，就可以直接执行这几个命令了

  + 解决方案3

    需要哪个命令将哪个命令放到/usr/bin中，使用

    ``` bash
    ln -s /home/prod/redis/redis-4.0.8/src/redis-server /usr/bin/redis-server
    ```

    相当于创建一个全局快捷方式

  注意：/usr/bin和/usr/local/bin的区别

+ 验证启动

  ``` 
  ps -ef | grep redis
  ```

  或者生成客户端

  ![image-20200311154312738](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200311154312738.png)

### 七、理解bind

bind配置了什么ip，别人就得访问bind里面配置的ip才访问到redis服务

可以绑定多个ip

bind ip1 ip2

#### 外网访问

+ 设置密码： vim  redis.conf     requirepass xxxx
+ 需要允许外网访问，则将redis.conf中修改   将bind注释掉
+ 设置protected-mode no

### 八、防火墙开放端口

> 1. 开放端口
>
> ```shell
> firewall-cmd --zone=public --add-port=6379/tcp --permanent
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

### 九、停止

``` 
#关闭服务器
redis-cli shutdown
```

不建议用kill -9 进程号来停止