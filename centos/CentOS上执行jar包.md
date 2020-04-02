## CentOS上执行jar包

### 一、命令后缀

+ 指定运行环境

  ```shell
  java -jar *.jar --spring.profiles.active=**
  ```

+ 指定port

  ```shell
  java -jar *.jar --server.port=****
  ```

### 二、后台运行

+ 方式一：

  ```shell
  java -jar *.jar
  ```


  特点：当前ssh窗口被锁定，可按CTRL + C打断程序运行，或直接关闭窗口，程序退出

  那如何让窗口不锁定？

+ 方式二

  ```shell
  java -jar *.jar &
  ```


  &代表在后台运行。

  特定：当前ssh窗口不被锁定，但是当窗口关闭时，程序中止运行。

  继续改进，如何让窗口关闭时，程序仍然运行？

+ 方式三

  ```shell
  nohup java -jar *.jar &	
  ```

  nohup 意思是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行

  当用 nohup 命令执行作业时，缺省情况下该作业的所有输出被重定向到nohup.out的文件中，除非另外指定了	输出文件。

+ 方式四

  ```shell
  nohup java -jar *.jar >temp.log &
  ```

  解释下 >temp.log

  command >out.file

  command >out.file是将command的输出重定向到out.file文件，即输出内容不打印到屏幕上，而是输出到out.file文件中

  但是会产生nohup: redirecting stderr to stdout问题，现象是控制台输出的信息一部分输出到了我指定的文件，另一部分却输出到了nohup.out，而我是不想让它产生nohup.out文件，不知道是什么原因。

+ 方式五

  ```shell
  nohup java -jar *.jar >temp.log 2>&1 &
  ```

  解释如下：

  2>
  表示把标准错误(stderr)重定向，标准输出(stdout)是1。

  尖括号后面可以跟文件名，或者是&1, &2，分别表示重定向到标准输出和标准错误。

  2> &1
  1> &2
  2> stderr.log
  1> stdout.log

### 三、操作运行任务

+ jobs ：查看当前有多少在后台执行的命令（包含正在运行或暂停的命令），那么就会列出所有后台执行的作业，并且每个作业前面都有个编号

+ ctrl + z ：将一个正在前台执行的命令放到后台，并且暂停

+ bg %num  ：将一个在后台暂停的命令继续执行

+ fg %num   ：将一个在后台执行的命令调至前台继续执行，将某个作业调回前台控制

+ kill %num  ：杀死在后台执行的命令

+ 查看某端口占用的线程的pid

  netstat -nlp |grep :9181

PS： num是指工作号，注意区分pid