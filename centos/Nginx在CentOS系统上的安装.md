## Nginx在CentOS系统上的安装

### 一、安装准备环境

+ gcc 

  对源码进行编译，依赖gcc环境，直接 yum install gcc-c++ 就行了

+ pcre

  是一个正则表达式库，yum install -y pcre pcre-devel 就行了

+ zlib

  zlib库主要用来压缩与解压缩，yum install -y zlib zlib-devel

+ openssl 

  看名字就知道开启ssl协议的 yum install -y openssl openssl-devel

### 二、安装nginx

#### 1. 解压tar包

解压到/usr/local文件夹下

```shell
tar -zxvf **.tar.gz -C /usr/local
```

这个和cp命令有点不同，cp命令如果不存在这个目录就会自动创建这个目录，前提要保证存在这个目录

#### 2. configure检查

进入解压的文件夹（比如nginx-1.16.1），然后

```shell
./configure
```

#### 3. 编译安装

检查完成后，输入

```shell
make && make install
```

#### 4. 启动

安装完成后，/usr/local中会有文件夹nginx

进入nginx，启动命令在/sbin下放着，输入如下命令启动

```shell
./nginx
```

### 三、动静分离

#### 1. 准备图片文件等

在/opt或者/home下放置，根据需要可以新建文件夹进行整理

#### 2. 修改配置文件

进入/usr/local/nginx/conf，打开nginx.conf配置文件

+ 先找到监听80端口的位置，将server_name由localhost改成外界需要访问的ip地址，不改也行......

+ 静态资源调整需要在location作用域进行

  location关键字后面加路由规则，root关键字后面加需要访问的上级文件夹路径，设置autoindex为on，表示可以列出当前目录下所有文件，比如：

  + 图片文件存放在/home/data/infer和/home/data/interview文件夹中

  + location需要这么写

    ![image-20200403104028716](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200403104028716.png)

  + 浏览器访问网址端口加/infer/或/interview/

    ![image-20200403104225917](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200403104225917.png)

#### 3. 改完之后需要重启nginx

+ 重新./nginx就行

  ./nginx 快速停止命令 ./nginx -s stop

  完整停止命令 ./nginx -s quit(好一点)

  重启命令 ./nginx -s reload

+ service nginx restart

  如果出现无法重启的情况，可以先   ps -ef | grep nginx 获取nginx的运行线程，然后kill -9 端口 杀死对应线程，然后在 service nginx start/stop/restart   启动/停止/重启

#### 4. 网上有种配置仅供参考

> location ~ .*\.(gif|jpg|jpeg|png)$ { 
>     expires 24h; 
>       root /home/images/;#指定图片存放路径 
>       access_log /usr/local/websrv/nginx-1.9.4/logs/images.log;#日志存放路径 
>       proxy_store on; 
>       proxy_store_access user:rw group:rw all:rw; 
>       proxy_temp_path     /home/images/;#图片访问路径 
>       proxy_redirect     off; 
>       proxy_set_header    Host 127.0.0.1; 
>       client_max_body_size  10m; 
>       client_body_buffer_size 1280k; 
>       proxy_connect_timeout  900; 
>       proxy_send_timeout   900; 
>       proxy_read_timeout   900; 
>       proxy_buffer_size    40k; 
>       proxy_buffers      40 320k; 
>       proxy_busy_buffers_size 640k; 
>       proxy_temp_file_write_size 640k; 
>       if ( !-e $request_filename) 
>       { 
>          proxy_pass http://127.0.0.1;#默认80端口 
>       } 
>      }   

