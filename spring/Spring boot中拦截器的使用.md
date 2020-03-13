## Spring boot中拦截器的使用

### 一、简介

Spring MVC 中的拦截器（Interceptor）类似于 Servlet 开发中的过滤器 Filter，它主要用于拦截用户请求并作相应的处理,它也是 AOP 编程思想的体现,底层通过动态代理模式完成。

我的理解：拦截器与AOP的参数和返回值都不同

+ 参数：拦截器的参数是http的request和response，可以拿到http参数，AOP的参数是可以取到Controller的参数
+ 返回值：拦截器的返回值是boolean（只有preHandle方法是boolean），它的true和false决定了controller会不会被执行，而AOP的返回值只能是void，是在controller的前、中、后期去做一些无关的事。

### 二、定义拦截器

拦截器有两种实现方式： 

1. 实现 HandlerInterceptor 接口 

2. 继承 HandlerInterceptorAdapter 抽象类（看源码最底层也是通过 HandlerInterceptor 接口 实现）

HandlerInterceptor接口都是default方法，不完全实现内部函数也可以

内部函数：

+ preHandle：在业务处理器处理请求之前被调用。预处理，可以进行编码、安全控制、权限校验等处理；

+ postHandle：在业务处理器处理请求执行完成后，生成视图之前执行。后处理（调用了Service并返回ModelAndView，但未进行页面渲染），有机会修改ModelAndView；

+ afterCompletion：在DispatcherServlet完全处理完请求后被调用，可用于清理资源等。返回处理（已经渲染了页面）；

```java
package edu.qingtai.poandse.interceptor;

import edu.qingtai.poandse.util.RedisUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.servlet.HandlerInterceptor;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class VerifyInterceptor implements HandlerInterceptor {
    @Autowired
    private RedisUtils redisUtils;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String rd3session = request.getHeader("sessionid");

        if(rd3session != null){
            if(redisUtils.get(rd3session) == null){
                response.getWriter().write("login");
                return false;
            }else{
                return true;
            }
        }else{
            response.getWriter().write("no sessionid");
            return false;
        }
    }
}
```

### 三、注入拦截器

规则：

+ 在旧版中，一般继承 WebMvcConfigurerAdapter类，但由于2.0后，前者已经过时，在spring boot2.x中，WebMvcConfigurerAdapter被deprecated，虽然继承WebMvcConfigurerAdapter这个类虽然有此便利，但在Spring5.0里面已经deprecated了。 官方文档也说了，WebMvcConfigurer接口现在已经有了默认的空白方法，所以在Springboot2.0（Spring5.0）下更好的做法还是implements WebMvcConfigurer

+ addPathPatterns 用来设置拦截路径，excludePathPatterns 用来设置白名单，也就是不需要触发这个拦截器的路径

  ```java
  registry.addInterceptor(new Test1Interceptor()) // 添加拦截器
  				.addPathPatterns("/**") // 添加拦截路径
  				.excludePathPatterns(// 添加排除拦截路径
  						"/hello").order(0);//执行顺序
  //第二个拦截器可以设置为.order(1)
  ```

  

+ addResourceHandler方法是设置访问路径前缀，addResourceLocations方法设置资源路径，如果你想指定外部的目录也很简单，直接addResourceLocations指定即可，代码如下：

  ```
  registry.addResourceHandler("/static/**").addResourceLocations("file:E:/cxy/");
  ```

+ 拦截器的运行顺序：如果设置两个拦截器，执行流程是先进后出

  ``` 
  执行Test1Interceptor preHandle方法-->01
  执行Test2Interceptor preHandle方法-->01
  这里是Test2
  执行Test2Interceptor postHandle方法-->02
  执行Test1Interceptor postHandle方法-->02
  执行Test2Interceptor afterCompletion方法-->03
  执行Test1Interceptor afterCompletion方法-->03
  ```

示例：

```java
package edu.qingtai.poandse.config;

import edu.qingtai.poandse.interceptor.VerifyInterceptor;
import edu.qingtai.poandse.util.RedisUtils;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Bean
    public VerifyInterceptor getVerifyInterceptor(){
        return new VerifyInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry){
        registry.addInterceptor(getVerifyInterceptor())
                .addPathPatterns("/**");
    }
}
```

### 四、更多介绍

可以参考[这里](https://juejin.im/post/5df4f5536fb9a0166243623c)

#### 1. 重申作用

在 **Spring中**，当请求发送到 **Controller** 时，在被**Controller**处理之前，它必须经过 **Interceptors**（0或多个）

+ 日志记录：记录请求信息的日志，以便进行信息监控、信息统计、计算 PV（Page View）等；
+ 权限检查：如登录检测，进入处理器检测是否登录；
+ 性能监控：通过拦截器在进入处理器之前记录开始时间，在处理完后记录结束时间，从而得到该请求的处理时间。（反向代理，如 Apache 也可以自动记录）
+ 通用行为：读取 Cookie 得到用户信息并将用户对象放入请求，从而方便后续流程使用，还有如提取 Locale、Theme 信息等，只要是多个处理器都需要的即可使用拦截器实现。

#### 2. 细看函数

1. `preHandler(HttpServletRequest request, HttpServletResponse response, Object handler)` 方法在请求处理之前被调用。该方法在 Interceptor 类中最先执行，用来进行一些前置初始化操作或是对当前请求做预处理，也可以进行一些判断来决定请求是否要继续进行下去。该方法的返回至是 Boolean 类型，当它返回 false 时，表示请求结束，后续的 Interceptor 和 Controller 都不会再执行；当它返回为 true 时会继续调用下一个 Interceptor 的 preHandle 方法，如果已经是最后一个 Interceptor 的时候就会调用当前请求的 Controller 方法。
2. `postHandler(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)` 方法在当前请求处理完成之后，也就是 Controller 方法调用之后执行，但是它会在  DispatcherServlet  进行视图返回渲染之前被调用，所以我们可以在这个方法中对 Controller 处理之后的 ModelAndView 对象进行操作。
3. `afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handle, Exception ex)` 方法需要在当前对应的 Interceptor 类的 preHandle 方法返回值为 true 时才会执行。顾名思义，该方法将在整个请求结束之后，也就是在 DispatcherServlet  渲染了对应的视图之后执行。此方法主要用来进行资源清理。

#### 3. 运行流程

![image-20200313205007244](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200313205007244.png)

![image-20200313205032712](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200313205032712.png)

![image-20200313211106515](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200313211106515.png)

