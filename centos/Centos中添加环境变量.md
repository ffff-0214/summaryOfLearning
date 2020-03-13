## Centos中添加环境变量

### 一、PATH

+ 环境变量其实就是PATH
+ 简单说PATH就是一组路径的字符串变量，当你输入的命令不带任何路径时，LINUX会在PATH记录的路径中查找该命令，有的话则执行，不存在则提示命令找不到
+ 比如在根目录/下可以输入命令ls,在/usr目录下也可以输入ls,但其实ls命令根本不在这个两个目录下，当你输入ls命令时LINUX会去/bin,/usr/bin,/sbin等目录寻找该命令。而PATH就是定义/bin:/sbin:/usr/bin等这些路劲的变量，其中冒号为目录间的分割符。

### 二、自定义路径

 假设你新编译安装了一个apache在/usr/local/apache下，你希望每次启动的时候不用敲一大串字符（# /usr/local/apache/bin/apachectl start）才能使用它，而是直接像ls一样在任何地方都直接输入类似这样（# apachectl start）的简短命令。这时，你就需要修改环境变量PATH了，准确的说就是给PATH增加一个值/usr/local/apache/bin。将/usr/local/apache/bin添加到PATH中有三种方法：

1. 直接在命令行中设置PATH

   ```bash
   $ PATH=$PATH:/usr/local/apache/bin
   ```

   使用这种方法,只对当前会话有效，也就是说每当登出或注销系统以后，PATH设置就会失效。

2. 在profile中设置PATH

   ```bash
   $ vim /etc/profile
   ```

   找到export行，在下面新增加一行，内容为：

   ``` 
   export PATH=$PATH:/usr/local/apache/bin
   ```

   + ＝ 等号两边不能有任何空格。这种方法最好,除非手动强制修改PATH的值,否则将不会被改变

   + 编辑/etc/profile后PATH的修改不会立马生效，如果需要立即生效的话，可以执行

     ```bash
     $ source profile
     ```

3. 在当前用户的profile中设置PATH

   ```bash
    $ vim ~/.bash_profile
   ```

   修改PATH行,把/usr/local/apache/bin添加进去,如：

   ```
   PATH=$PATH:$HOME/bin:/usr/local/apache/bin
   ```

   ```bash
   $source ~/.bash_profile
   ```

      让这次的修改生效，这种方法只对当前用户起作用的,其他用户该修改无效。 

+ 当你发现新增路径/usr/local/apache/bin没用或不需要时，你可以在以前修改的/etc/profile或~/.bash_profile文件中删除你曾今自定义的路径

### 三、问题

为什么每次进入命令都要重新source /etc/profile 才能生效？

关机之后重启就发现环境变量竟然没有生效，还得使用source命令

#### 1. 分析

命令行中输入以下命令

```
 cat ~/.bash_profile
```

结果如下

```bash
# .bash_profile 

# Get the aliases and functions
if [ -f ~/.bashrc ]; then  
. ~/.bashrc
fi
# User specific environment and startup programs
PATH=$PATH:$HOME/bin 
export PATH
```

继续看看~/.bashrc中都有什么

```bash
# .bashrc
 
# User specific aliases and functions
 
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
 
# Source global definitions
if [ -f /etc/bashrc ]; then
    . /etc/bashrc
fi
```

继续看看/etc/bashrc 中都有什么，结果发现，环境变量在这里面设置的，于是像上面讲的那样，把环境变量加进去，然后 source /etc/bashrc，每次登录都可以了

#### 2.解决

+ 将环境变量放在~/.bashrc里面
+ 或在~/.bashrc里面加一句source /etc/profile
+ 或在/etc/bashrc中设置环境变量

