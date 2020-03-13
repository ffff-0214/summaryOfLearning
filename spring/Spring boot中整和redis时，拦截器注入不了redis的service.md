## Spring boot中整和redis时，拦截器注入不了redis的service

详细内容在[这里](https://blog.csdn.net/liuyang1835189/article/details/81056162)

+ Spring在容器加载的时候会根据注解和xml创建各种bean，如果是注解没写bean的作用域默认是单例模式，

> 单例模式的作用域：单例模式，Spring IoC容器中只会存在一个共享的Bean实例，无论有多少个Bean引用它，始终指向同一对象。Singleton作用域是Spring中的缺省作用域，也可以显示的将Bean定义为singleton模式。

+ 在配置拦截器时，我们会在add函数中new一个拦截器实例，spring boot容器在启动的时候，会默认扫描application所在的包的注解，也扫描了autowired注入的redis服务这个拦截器暂且给它命名个名字对象A,当发生登陆请求的时候，首先会经过拦截器，在拦截器栈里是new了一个拦截器对象，所以这个对象并不是A,所以当打入断点调入这个拦截器的时候，再注入redis服务，所以才会空，**bean注入只在容器初始化的时候注入**。那么我在这个拦截器上面加个注解Component注解可不可以了，是不可以的，Spring boot在扫描的时候虽然能扫描到，但是注入的对象和上面情况一样，不是同一bean，

+ `@Component` 注解并没有通过 cglib 来代理`@Bean` 方法的调用，因此new实例这样配置时，就是不同的实例

+ 最好的方式就是通过@bean的方式注入bean，不能通过new 创建一个对象了，直接调用上面的get方法

![image-20200313210116593](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200313210116593.png)

Springboot在初始化容器的时候通过注解的扫描创建了共享bean实例，在发生url请求拦截的时候，**不会在创建新的对象**，从容器中取了已经注入服务的bean