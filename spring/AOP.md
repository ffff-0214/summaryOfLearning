## AOP

### 一、思想

#### 1. 面向对象编程

面向对象编程，也称为OOP（即Object Oriented Programming）最大的优点在于能够将业务模块进行封装，从而达到功能复用的目的。通过面向对象编程，不同的模板可以相互组装，从而实现更为复杂的业务模块，其结构形式可用下图表示：

![业务模块](https://upload-images.jianshu.io/upload_images/7944306-5b43b08601ddc9c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

​    面向对象编程解决了业务模块的封装复用的问题，但是对于某些模块，其本身并不独属于摸个业务模块，而是根据不同的情况，贯穿于某几个或全部的模块之间的。例如登录验证，其只开放几个可以不用登录的接口给用户使用（一般登录使用拦截器实现，但是其切面思想是一致的）；再比如性能统计，其需要记录每个业务模块的调用，并且监控器调用时间。可以看到，这些横贯于每个业务模块的模块，如果使用面向对象的方式，那么就需要在已封装的每个模块中添加相应的重复代码，对于这种情况，面向切面编程就可以派上用场了。

#### 2. 面向切面编程

面向切面编程，也称为AOP（即Aspect Oriented Programming），指的是将一定的切面逻辑按照一定的方式编织到指定的业务模块中，从而将这些业务模块的调用包裹起来。如下是其结构示意图：

![切面编程模块](https://upload-images.jianshu.io/upload_images/7944306-64afabb49a2cd503.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3. 综述

+ OOP（面向对象编程）通过的是继承、封装和多态等概念来建立一种对象层次结构，用于模拟公共行为的一个集合。OOP从纵向上区分出一个个的类来，而AOP则从横向上向对象中加入特定的代码。AOP使OOP由原来的二维变为三维了，由平面变成立体了。

+ AOP采用"横切"的技术，剖解开封装的对象内部，将影响了多个类的公共行为封装到一个可重用模块。将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

+ 简单来说讲，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。

#### 4. 理解

为什么说AOP是OOP的补充和完善呢？

+ 如果仅仅为了重用通用的功能，OOP中继承或委托也可以完成。但是如果整个应用中都使用相同的基类，继承往往会导致一个脆弱的对象体系，难以修改维护。而使用委托则会需要委托对象进行复杂的调用。

+ 而AOP提供了取代继承和委托的另一种可选方案，而且更加清晰明了。在使用面向切面编程时，我们仍然需要在定义一个通用功能，但是可以通过声明的方式定义这个功能以何种方式在何处应用，而不需要改变受影响的类。横向关注点可以被模块化为特殊的类，这些类被称为切面。这样做有两个好处：每个关注点都集中于一个地方而不是分散在多处代码中

+ 优点

  1、面向切面编程使得每个关注点都集中于一个地方而不是分散在多处代码中，便于后期的统一维护管理。

  2、服务模块更简洁，它们只包含主要关注点，而次要关注点的代码被转移到切面中了。

  3、对原方法进行方法增强，且不影响原方法的正常使用。

  4、使用简单可插拔的配置，在实际逻辑执行之前、之后或周围动态添加横切关注点。

#### 5. 应用场景举例

1. 日志模块

   日志代码往往横向地散布在所有对象层次中，而与它对应的对象的核心功能毫无关系对于其他类型的代码，如安全性、异常处理和透明的持续性也都是如此。

2. 事务管理

   调用方法前开启事务， 调用方法后提交关闭事务

   ![image-20200304211846103](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200304211846103.png)

#### 6. AOP术语

![image-20200304212011906](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200304212011906.png)

1） 切面（Aspect）
切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——它是什么，在何时和何处完成其功能。比如事务管理是一个切面，权限管理也是一个切面。

2） 通知（Advice）
通知定义了切面是什么以及何时使用。除了描述切面要完成的工作，通知还解决了何时执行这个工作的问题。
Spring切面可以应用5种类型的通知：
前置通知（Before）：在目标方法被调用之前调用通知功能
后置通知（After）：在目标方法完成之后调用通知，不关心方法的输出是什么。是“返回通知”和“异常通知”的并集。
返回通知（After-returning）：在目标方法成功执行之后调用通知
异常通知（After-throwing）：在目标方法抛出异常后调用通知
环绕通知（Around）通知包裹了被通知的方法，可同时定义前置通知和后置通知。

3） 切点（Pointcut）
切点定义了在何处工作，也就是真正被切入的地方，也就是在哪个方法应用通知。切点的定义会匹配通知所有要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。

4）连接点（Join point）
连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时，抛出异常时，甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

5） 引入（Introduction）
引介让一个切面可以声明被通知的对象实现了任何他们没有真正实现的额外接口，而且为这些对象提供接口的实现。
引入允许我们向现有的类添加新方法或属性。这个新方法和实例变量就可以被引入到现有的类中，从而可以再无需修改这些现有的类的情况下，让它们具有新的行为和状态。

5） 织入（Weaving）：织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以织入。
编译器：切面在目标类编译时被织入。这种方式需要特殊的编译器。
类加载期：切面在目标类被引入应用之前增强该目标类的字节码。
运行期：切面在应用运行的某个时刻被织入。

### 二、编写

基于SpringBoot编写了一个简单的Spring AOPDemo

#### 1. 依赖

```xml
maven依赖添加如下
<!--引入SpringBoot的Web模块-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
 
<!--引入AOP依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

注意：在完成了引入AOP依赖包后，不需要去做其他配置。AOP的默认配置属性中，spring.aop.auto属性默认是开启的，也就是说只要引入了AOP依赖后，默认已经增加了`@EnableAspectJAutoProxy`，不需要在程序主类中增加`@EnableAspectJAutoProxy`来启用。

#### 2. 业务模块

web请求入口：

```java
package com.example.demo.aop;
 
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
/**
* @desc: 核心业务模块
* @author: CSH
**/
@RestController
@RequestMapping("/aopController")
public class AopController {
 
    @RequestMapping(value = "/Curry")
    public void Curry(){
        System.out.println("库里上场打球了！！");
    }
 
    @RequestMapping(value = "/Harden")
    public void Harden(){
        System.out.println("哈登上场打球了！！");
    }
    
    @RequestMapping(value = "/Antetokounmpo")
    public void Antetokounmpo(){
        System.out.println("字母哥上场打球了！！");
    }
 
    @RequestMapping(value = "/Jokic")
    public void Jokic(){
        System.out.println("约基奇上场打球了！！");
    }
 
    @RequestMapping(value = "/Durant/{point}")
    public void Durant(@PathVariable("point")  int point){
        System.out.println("杜兰特上场打球了！！");
    }
}
```

定义切面类：在类上添加@Aspect 和@Component 注解即可将一个类定义为切面类。

@Aspect 注解 使之成为切面类

@Component 注解 把切面类加入到IOC容器中

```java
package com.example.demo.aop;
 
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;
 
/**
* @desc: 经纪人切面
* @author: CSH
**/
@Aspect
@Component
public class BrokerAspect {
 
    /**
     * 定义切入点，切入点为com.example.demo.aop.AopController中的所有函数
     *通过@Pointcut注解声明频繁使用的切点表达式
     */
    @Pointcut("execution(public * com.example.demo.aop.AopController.*(..)))")
    public void BrokerAspect(){
 
    }
 
    /**
    * @description  在连接点执行之前执行的通知
    */
    @Before("BrokerAspect()")
    public void doBeforeGame(){
        System.out.println("经纪人正在处理球星赛前事务！");
    }
 
    /**
     * @description  在连接点执行之后执行的通知（返回通知和异常通知的异常）
     */
    @After("BrokerAspect()")
    public void doAfterGame(){
        System.out.println("经纪人为球星表现疯狂鼓掌！");
    }
 
    /**
     * @description  在连接点执行之后执行的通知（返回通知）
     */
    @AfterReturning("BrokerAspect()")
    public void doAfterReturningGame(){
        System.out.println("返回通知：经纪人为球星表现疯狂鼓掌！");
    }
 
    /**
     * @description  在连接点执行之后执行的通知（异常通知）
     */
    @AfterThrowing("BrokerAspect()")
    public void doAfterThrowingGame(){
        System.out.println("异常通知：球迷要求退票！");
    }
}
```

调用服务，输出结果：

![image-20200304213016762](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200304213016762.png)

切点表达式用于描述切点的位置信息，在此简单描述文中切点表达式的含义：

![image-20200304213114898](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200304213114898.png)

此[博客](https://www.cnblogs.com/zhangxufeng/p/9160869.html)详细讲解了切点表达式

#### 3. 环绕通知

环绕通知可以将你所编写的逻辑将被通知的目标方法完全包装起来。我们可以使用一个环绕通知来代替之前多个不同的前置通知和后置通知。如下所示，前置通知和后置通知位于同一个方法中，不像之前那样分散在不同的通知方法里面

```java
/**
* @description  使用环绕通知
*/
@Around("BrokerAspect()")
public void doAroundGame(ProceedingJoinPoint pjp) throws Throwable {
    try{
        System.out.println("经纪人正在处理球星赛前事务！");
        pjp.proceed();
        System.out.println("返回通知：经纪人为球星表现疯狂鼓掌！");
    }
    catch(Throwable e){
        System.out.println("异常通知：球迷要求退票！");
    }
}
```

环绕通知接受ProceedingJoinPoint作为参数，它来调用被通知的方法。通知方法中可以做任何的事情，当要将控制权交给被通知的方法时，需要调用ProceedingJoinPoint的proceed()方法。当你不调用proceed()方法时，将会阻塞被通知方法的访问

#### 4. 传参处理

当通知方法需要传入参数时，和之前创建的切面一样，这里的不同点在于切点还声明了要提供给通知方法的参数。切点表达式args(point)表明传递给GameDataAspect()方法中的int类型参数也会传递到通知中去，参数名point和缺点方法签名中的参数相匹配

```java
package com.example.demo.aop;
 
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;
 
/**
* @desc:技术统计
* @author: CSH
**/
@Aspect
@Component
public class GameDataAspect {
    /**
     * 定义切入点，切入点为com.example.demo.aop.AopController中的所有函数
     *通过@Pointcut注解声明频繁使用的切点表达式
     */
    @Pointcut("execution(public * com.example.demo.aop.AopController.Durant(int)) && args(point))")
    public void GameDataAspect(int point){
 
    }
 
    /**
     * @description  使用环绕通知
     */
    @Around("GameDataAspect(point)")
    public void doAroundGameData(ProceedingJoinPoint pjp,int point) throws Throwable {
        try{
            System.out.println("球星上场前热身！");
            pjp.proceed();
            System.out.println("球星本场得到" + point + "分" );
        }
        catch(Throwable e){
            System.out.println("异常通知：球迷要求退票！");
        }
    }
}
```

调用服务，输出结果：

![image-20200304213648065](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200304213648065.png)

### 三、表达式总结

#### 1. Advice

Advice：也即切面逻辑，指定了当前用于包裹切面指定的业务模块的逻辑

类型如下：

- @Before：该注解标注的方法在业务模块代码执行之前执行，其不能阻止业务模块的执行，除非抛出异常；
- @AfterReturning：该注解标注的方法在业务模块代码执行之后执行；
- @AfterThrowing：该注解标注的方法在业务模块抛出指定异常后执行；
- @After：该注解标注的方法在所有的Advice执行完成后执行，无论业务模块是否抛出异常，类似于finally的作用；
- @Around：该注解功能最为强大，其所标注的方法用于编写包裹业务模块执行的代码，其可以传入一个ProceedingJoinPoint用于调用业务模块的代码，无论是调用前逻辑还是调用后逻辑，都可以在该方法中编写，甚至其可以根据一定的条件而阻断业务模块的调用；
- @DeclareParents：其是一种Introduction类型的模型，在属性声明上使用，主要用于为指定的业务模块添加新的接口和相应的实现。
- @Aspect：严格来说，其不属于一种Advice，该注解主要用在类声明上，指明当前类是一个组织了切面逻辑的类，并且该注解中可以指定当前类是何种实例化方式，主要有三种：singleton、perthis和pertarget，具体的使用方式后面会进行讲解。

​    这里需要说明的是，@Before是业务逻辑执行前执行，与其对应的是@AfterReturning，而不是@After，@After是所有的切面逻辑执行完之后才会执行，无论是否抛出异常。

#### 2. 切点表达式

1）execution

 由于Spring切面粒度最小是达到方法级别，而execution表达式可以用于明确指定方法返回类型，类名，方法名和参数名等与方法相关的部件，并且在Spring中，大部分需要使用AOP的业务场景也只需要达到方法级别即可，因而execution表达式的使用是最为广泛的。如下是execution表达式的语法：

```java
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)
```

这里问号表示当前项可以有也可以没有，其中各项的语义如下：

- modifiers-pattern：方法的可见性，如public，protected；
- ret-type-pattern：方法的返回值类型，如int，void等；
- declaring-type-pattern：方法所在类的全路径名，如com.spring.Aspect；
- name-pattern：方法名类型，如buisinessService()；
- param-pattern：方法的参数类型，如java.lang.String；
- throws-pattern：方法抛出的异常类型，如java.lang.Exception；

如下是一个使用execution表达式的例子：

```java
execution(public * com.spring.service.BusinessObject.businessService(java.lang.String,..))
```

上述切点表达式将会匹配使用public修饰，返回值为任意类型，并且是com.spring.BusinessObject类中名称为businessService的方法，方法可以有多个参数，但是第一个参数必须是java.lang.String类型的方法。

上述示例中我们使用了..通配符，关于通配符的类型，主要有两种：

- *通配符，该通配符主要用于匹配单个单词，或者是以某个词为前缀或后缀的单词。

​    如下示例表示返回值为任意类型，在com.spring.service.BusinessObject类中，并且参数个数为零的方法：

```java
execution(* com.spring.service.BusinessObject.*())
```

​    下述示例表示返回值为任意类型，在com.spring.service包中，以Business为前缀的类，并且是类中参数个数为零方法：

```java
execution(* com.spring.service.Business*.*())
```

- ..通配符，该通配符表示0个或多个项，主要用于declaring-type-pattern和param-pattern中，如果用于declaring-type-pattern中，则表示匹配当前包及其子包，如果用于param-pattern中，则表示匹配0个或多个参数。

​    如下示例表示匹配返回值为任意类型，并且是com.spring.service包及其子包下的任意类的名称为businessService的方法，而且该方法不能有任何参数：

```java
execution(* com.spring.service..*.businessService())
```

​    这里需要说明的是，包路径service..\*.businessService()中的..应该理解为延续前面的service路径，表示到service路径为止，或者继续延续service路径，从而包括其子包路径；后面的*.businessService()，这里的\*表示匹配一个单词，因为是在方法名前，因而表示匹配任意的类。

​    如下示例是使用..表示任意个数的参数的示例，需要注意，表示参数的时候可以在括号中事先指定某些类型的参数，而其余的参数则由..进行匹配：

```java
execution(* com.spring.service.BusinessObject.businessService(java.lang.String,..))
```



2）within

within表达式的粒度为类，其参数为全路径的类名（可使用通配符），表示匹配当前表达式的所有类都将被当前方法环绕。如下是within表达式的语法：

```java
within(declaring-type-pattern)
```

​    within表达式只能指定到类级别，如下示例表示匹配com.spring.service.BusinessObject中的所有方法：

```java
within(com.spring.service.BusinessObject)
```

​    within表达式路径和类名都可以使用通配符进行匹配，比如如下表达式将匹配com.spring.service包下的所有类，不包括子包中的类：

```java
within(com.spring.service.*)
```

​    如下表达式表示匹配com.spring.service包及子包下的所有类：

```java
within(com.spring.service..*)
```



3）args

args表达式的作用是匹配指定参数类型和指定参数数量的方法，无论其类路径或者是方法名是什么。这里需要注意的是，args指定的参数必须是全路径的。如下是args表达式的语法：

```java
args(param-pattern)
```

​    如下示例表示匹配所有只有一个参数，并且参数类型是java.lang.String类型的方法：

```java
args(java.lang.String)
```

​    也可以使用通配符，但这里通配符只能使用..，而不能使用*。如下是使用通配符的实例，该切点表达式将匹配第一个参数为java.lang.String，最后一个参数为java.lang.Integer，并且中间可以有任意个数和类型参数的方法：

```java
args(java.lang.String,..,java.lang.Integer)
```



4）this和target

 this和target需要放在一起进行讲解，主要目的是对其进行区别。this和target表达式中都只能指定类或者接口，在面向切面编程规范中，this表示匹配调用当前切点表达式所指代对象方法的对象，target表示匹配切点表达式指定类型的对象。比如有两个类A和B，并且A调用了B的某个方法，如果切点表达式为this(B)，那么A的实例将会被匹配，也即其会被使用当前切点表达式的Advice环绕；如果这里切点表达式为target(B)，那么B的实例也即被匹配，其将会被使用当前切点表达式的Advice环绕。

​    在讲解Spring中的this和target的使用之前，首先需要讲解一个概念：业务对象（目标对象）和代理对象。对于切面编程，有一个目标对象，也有一个代理对象，目标对象是我们声明的业务逻辑对象，而代理对象是使用切面逻辑对业务逻辑进行包裹之后生成的对象。如果使用的是Jdk动态代理，那么业务对象和代理对象将是两个对象，在调用代理对象逻辑时，其切面逻辑中会调用目标对象的逻辑；如果使用的是Cglib代理，由于是使用的子类进行切面逻辑织入的，那么只有一个对象，即织入了代理逻辑的业务类的子类对象，此时是不会生成业务类的对象的。

​    在Spring中，其对this的语义进行了改写，即如果当前对象生成的代理对象符合this指定的类型，那么就为其织入切面逻辑。简单的说就是，this将匹配代理对象为指定类型的类。target的语义则没有发生变化，即其将匹配业务对象为指定类型的类。如下是使用this和target表达式的简单示例：

```java
this(com.spring.service.BusinessObject)
target(com.spring.service.BusinessObject)
```

通过上面的讲解可以看出，this和target的使用区别其实不大，大部分情况下其使用效果是一样的，但其区别也还是有的。Spring使用的代理方式主要有两种：Jdk代理和Cglib代理（关于这两种代理方式的讲解可以查看本人的文章[代理模式实现方式及优缺点对比](https://my.oschina.net/zhangxufeng/blog/1633187)）。针对这两种代理类型，关于目标对象与代理对象，理解如下两点是非常重要的：

- 如果目标对象被代理的方法是其实现的某个接口的方法，那么将会使用Jdk代理生成代理对象，此时代理对象和目标对象是两个对象，并且都实现了该接口；
- 如果目标对象是一个类，并且其没有实现任意接口，那么将会使用Cglib代理生成代理对象，并且只会生成一个对象，即Cglib生成的代理类的对象。

​    结合上述两点说明，这里理解this和target的异同就相对比较简单了。我们这里分三种情况进行说明：

- this(SomeInterface)或target(SomeInterface)：这种情况下，无论是对于Jdk代理还是Cglib代理，其目标对象和代理对象都是实现SomeInterface接口的（Cglib生成的目标对象的子类也是实现了SomeInterface接口的），因而this和target语义都是符合的，此时这两个表达式的效果一样；
- this(SomeObject)或target(SomeObject)，这里SomeObject没实现任何接口：这种情况下，Spring会使用Cglib代理生成SomeObject的代理类对象，由于代理类是SomeObject的子类，子类的对象也是符合SomeObject类型的，因而this将会被匹配，而对于target，由于目标对象本身就是SomeObject类型，因而这两个表达式的效果一样；
- this(SomeObject)或target(SomeObject)，这里SomeObject实现了某个接口：对于这种情况，虽然表达式中指定的是一种具体的对象类型，但由于其实现了某个接口，因而Spring默认会使用Jdk代理为其生成代理对象，Jdk代理生成的代理对象与目标对象实现的是同一个接口，但代理对象与目标对象还是不同的对象，由于代理对象不是SomeObject类型的，因而此时是不符合this语义的，而由于目标对象就是SomeObject类型，因而target语义是符合的，此时this和target的效果就产生了区别；这里如果强制Spring使用Cglib代理，因而生成的代理对象都是SomeObject子类的对象，其是SomeObject类型的，因而this和target的语义都符合，其效果就是一致的。

​    关于this和target的异同，我们使用如下示例进行简单演示：

```java
// 目标类
public class Apple {
  public void eat() {
    System.out.println("Apple.eat method invoked.");
  }
}
// 切面类
@Aspect
public class MyAspect {
  @Around("this(com.business.Apple)")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
    System.out.println("this is before around advice");
    Object result = pjp.proceed();
    System.out.println("this is after around advice");
    return result;
  }
}
<!-- bean声明文件 -->
<bean id="apple" class="chapter7.eg1.Apple"/>
<bean id="aspect" class="chapter7.eg6.MyAspect"/>
<aop:aspectj-autoproxy/>
// 驱动类
public class AspectApp {
  public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    Apple fruit = (Apple) context.getBean("apple");
    fruit.eat();
  }
}
```

​    执行驱动类中的main方法，结果如下：

```
this is before around advice
Apple.eat method invoked.
this is after around advice
```

​    上述示例中，Apple没有实现任何接口，因而使用的是Cglib代理，this表达式会匹配Apple对象。这里将切点表达式更改为target，还是执行上述代码，会发现结果还是一样的：

```java
target(com.business.Apple)
```

​    如果我们对Apple的声明进行修改，使其实现一个接口，那么这里就会显示出this和target的执行区别了：

```java
public class Apple implements IApple {
  public void eat() {
    System.out.println("Apple.eat method invoked.");
  }
}
public class AspectApp {
  public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    Fruit fruit = (Fruit) context.getBean("apple");
    fruit.eat();
  }
}
```

​    我们还是执行上述代码，对于this表达式，其执行结果如下：

```java
Apple.eat method invoked.
```

​    对于target表达式，其执行结果如下：

```java
this is before around advice
Apple.eat method invoked.
this is after around advice
```

​    可以看到，这种情况下this和target表达式的执行结果是不一样的，这正好符合我们前面讲解的第三种情况。



5）@within

 前面我们讲解了within的语义表示匹配指定类型的类实例，这里的@within表示匹配带有指定注解的类，其使用语法如下所示：

```java
@within(annotation-type)
```

​    如下所示示例表示匹配使用com.spring.annotation.BusinessAspect注解标注的类：

```java
@within(com.spring.annotation.BusinessAspect)
```

​    这里我们使用一个例子演示@within的用法（这里驱动类和xml文件配置与3.4节使用的一致，这里省略）：

```java
// 注解类
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface FruitAspect {
}
// 目标类
@FruitAspect
public class Apple {
  public void eat() {
    System.out.println("Apple.eat method invoked.");
  }
}
// 切面类
@Aspect
public class MyAspect {
  @Around("@within(com.business.annotation.FruitAspect)")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
    System.out.println("this is before around advice");
    Object result = pjp.proceed();
    System.out.println("this is after around advice");
    return result;
  }
}
```

​    上述切面表示匹配使用FruitAspect注解的类，而Apple则使用了该注解，因而Apple类方法的调用会被切面环绕，执行运行驱动类可得到如下结果，说明Apple.eat()方法确实被环绕了：

```
this is before around advice
Apple.eat method invoked.
this is after around advice
```



6）@annotation

@annotation的使用方式与@within的相似，表示匹配使用@annotation指定注解标注的方法将会被环绕，其使用语法如下：

```java
@annotation(annotation-type)
```

​    如下示例表示匹配使用com.spring.annotation.BusinessAspect注解标注的方法：

```java
@annotation(com.spring.annotation.BusinessAspect)
```

​    这里我们继续复用3.5节使用的例子进行讲解@annotation的用法，只是这里需要对Apple和MyAspect使用和指定注解的方式进行修改，FruitAspect不用修改的原因是声明该注解时已经指定了其可以使用在类，方法和参数上：

```java
// 目标类，将FruitAspect移到了方法上
public class Apple {
  @FruitAspect
  public void eat() {
    System.out.println("Apple.eat method invoked.");
  }
}
@Aspect
public class MyAspect {
  @Around("@annotation(com.business.annotation.FruitAspect)")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
    System.out.println("this is before around advice");
    Object result = pjp.proceed();
    System.out.println("this is after around advice");
    return result;
  }
}
```

​    这里Apple.eat()方法使用FruitAspect注解进行了标注，因而该方法的执行会被切面环绕，其执行结果如下：

```
this is before around advice
Apple.eat method invoked.
this is after around advice
```



7）@args

@within和@annotation分别表示匹配使用指定注解标注的类和标注的方法将会被匹配，@args则表示使用指定注解标注的类作为某个方法的参数时该方法将会被匹配。如下是@args注解的语法：

```java
@args(annotation-type)
```

​    如下示例表示匹配使用了com.spring.annotation.FruitAspect注解标注的类作为参数的方法：

```java
@args(com.spring.annotation.FruitAspect)
```

​    这里我们使用如下示例对@args的用法进行讲解：

```java
<!-- xml配置文件 -->
<bean id="bucket" class="chapter7.eg1.FruitBucket"/>
<bean id="aspect" class="chapter7.eg6.MyAspect"/>
<aop:aspectj-autoproxy/>
// 使用注解标注的参数类
@FruitAspect
public class Apple {}
// 使用Apple参数的目标类
public class FruitBucket {
  public void putIntoBucket(Apple apple) {
    System.out.println("put apple into bucket.");
  }
}
@Aspect
public class MyAspect {
  @Around("@args(chapter7.eg6.FruitAspect)")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
    System.out.println("this is before around advice");
    Object result = pjp.proceed();
    System.out.println("this is after around advice");
    return result;
  }
}
// 驱动类
public class AspectApp {
  public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    FruitBucket bucket = (FruitBucket) context.getBean("bucket");
    bucket.putIntoBucket(new Apple());
  }
}
```

​    这里FruitBucket.putIntoBucket(Apple)方法的参数Apple使用了@args注解指定的FruitAspect进行了标注，因而该方法的调用将会被环绕。执行驱动类，结果如下：

```
this is before around advice
put apple into bucket.
this is after around advice
```



8）@DeclareParents

@DeclareParents也称为Introduction（引入），表示为指定的目标类引入新的属性和方法。关于@DeclareParents的原理其实比较好理解，因为无论是Jdk代理还是Cglib代理，想要引入新的方法，只需要通过一定的方式将新声明的方法织入到代理类中即可，因为代理类都是新生成的类，因而织入过程也比较方便。如下是@DeclareParents的使用语法：

```java
@DeclareParents(value = "TargetType", defaultImpl = WeaverType.class)
private WeaverInterface attribute;
```

​    这里TargetType表示要织入的目标类型（带全路径），WeaverInterface中声明了要添加的方法，WeaverType中声明了要织入的方法的具体实现。如下示例表示在Apple类中织入IDescriber接口声明的方法：

```java
@DeclareParents(value = "com.spring.service.Apple", defaultImpl = DescriberImpl.class)
private IDescriber describer;
```

​    这里我们使用一个如下实例对@DeclareParents的使用方式进行讲解，配置文件与3.4节的一致，这里略：

```java
// 织入方法的目标类
public class Apple {
  public void eat() {
    System.out.println("Apple.eat method invoked.");
  }
}
// 要织入的接口
public interface IDescriber {
  void desc();
}
// 要织入接口的默认实现
public class DescriberImpl implements IDescriber {
  @Override
  public void desc() {
    System.out.println("this is an introduction describer.");
  }
}
// 切面实例
@Aspect
public class MyAspect {
  @DeclareParents(value = "com.spring.service.Apple", defaultImpl = DescriberImpl.class)
  private IDescriber describer;
}
// 驱动类
public class AspectApp {
  public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    IDescriber describer = (IDescriber) context.getBean("apple");
    describer.desc();
  }
}
```

​    在MyAspect中声明了我们需要将IDescriber的方法织入到Apple实例中，在驱动类中我们可以看到，我们获取的是apple实例，但是得到的bean却可以强转为IDescriber类型，因而说明我们的织入操作成功了。



9）perthis和pertarget

在Spring AOP中，切面类的实例只有一个，比如前面我们一直使用的MyAspect类，假设我们使用的切面类需要具有某种状态，以适用某些特殊情况的使用，比如多线程环境，此时单例的切面类就不符合我们的要求了。在Spring AOP中，切面类默认都是单例的，但其还支持另外两种多例的切面实例的切面，即perthis和pertarget，需要注意的是perthis和pertarget都是使用在切面类的@Aspect注解中的。这里perthis和pertarget表达式中都是指定一个切面表达式，其语义与前面讲解的this和target非常的相似，perthis表示如果某个类的代理类符合其指定的切面表达式，那么就会为每个符合条件的目标类都声明一个切面实例；pertarget表示如果某个目标类符合其指定的切面表达式，那么就会为每个符合条件的类声明一个切面实例。从上面的语义可以看出，perthis和pertarget的含义是非常相似的。如下是perthis和pertarget的使用语法：

```java
perthis(pointcut-expression)
pertarget(pointcut-expression)
```

​    由于perthis和pertarget的使用效果大部分情况下都是一致的，我们这里主要讲解perthis和pertarget的区别。关于perthis和pertarget的使用，需要注意的一个点是，由于perthis和pertarget都是为每个符合条件的类声明一个切面实例，因而切面类在配置文件中的声明上一定要加上prototype，否则Spring启动是会报错的。如下是我们使用的示例：

```java
<!-- xml配置文件 -->
<bean id="apple" class="chapter7.eg1.Apple"/>
<bean id="aspect" class="chapter7.eg6.MyAspect" scope="prototype"/>
<aop:aspectj-autoproxy/>
// 目标类实现的接口
public interface Fruit {
  void eat();
}
// 业务类
public class Apple implements Fruit {
  public void eat() {
    System.out.println("Apple.eat method invoked.");
  }
}
// 切面类
@Aspect("perthis(this(com.spring.service.Apple))")
public class MyAspect {

  public MyAspect() {
    System.out.println("create MyAspect instance, address: " + toString());
  }

  @Around("this(com.spring.service.Apple)")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
    System.out.println("this is before around advice");
    Object result = pjp.proceed();
    System.out.println("this is after around advice");
    return result;
  }
}
// 驱动类
public class AspectApp {
  public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    Fruit fruit = context.getBean(Fruit.class);
    fruit.eat();
  }
}
```

​    这里我们使用的切面表达式语法为perthis(this(com.spring.service.Apple))，这里this表示匹配代理类是Apple类型的类，perthis则表示会为这些类的每个实例都创建一个切面类。由于Apple实现了Fruit接口，因而Spring使用Jdk动态代理为其生成代理类，也就是说代理类与Apple都实现了Fruit接口，但是代理类不是Apple类型，因而这里声明的切面不会匹配到Apple类。执行上述驱动类，结果如下：

```
Apple.eat method invoked.
```

​    结果表明Apple类确实没有被环绕。如果我们讲切面类中的perthis和this修改为pertarget和target，效果如何呢：

```java
@Aspect("pertarget(target(com.spring.service.Apple))")
public class MyAspect {

  public MyAspect() {
    System.out.println("create MyAspect instance, address: " + toString());
  }

  @Around("target(com.spring.service.Apple)")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
    System.out.println("this is before around advice");
    Object result = pjp.proceed();
    System.out.println("this is after around advice");
    return result;
  }
}
```

​    执行结果如下：

```
create MyAspect instance, address: chapter7.eg6.MyAspect@48fa0f47
this is before around advice
Apple.eat method invoked.
this is after around advice
```

​    可以看到，Apple类被切面环绕了。这里target表示目标类是Apple类型，虽然Spring使用了Jdk动态代理实现切面的环绕，代理类虽不是Apple类型，但是目标类却是Apple类型，符合target的语义，而pertarget会为每个符合条件的表达式的类实例创建一个代理类实例，因而这里Apple会被环绕。

​    由于代理类与目标类的差别非常小，因而与this和target一样，perthis和pertarget的区别也非常小，大部分情况下其使用效果是一致的。关于切面多实例的创建，其演示比较简单，我们可以将xml文件中的Apple实例修改为prototype类型，并且在驱动类中多次获取Apple类的实例：

```java
<!-- xml配置文件 -->
<bean id="apple" class="chapter7.eg1.Apple" scope="prototype"/>
<bean id="aspect" class="chapter7.eg6.MyAspect" scope="prototype"/>
<aop:aspectj-autoproxy/>
public class AspectApp {
  public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    Fruit fruit = context.getBean(Fruit.class);
    fruit.eat();
    fruit = context.getBean(Fruit.class);
    fruit.eat();
  }
}
```

​    执行结果如下：

```
create MyAspect instance, address: chapter7.eg6.MyAspect@48fa0f47
this is before around advice
Apple.eat method invoked.
this is after around advice
create MyAspect instance, address: chapter7.eg6.MyAspect@56528192
this is before around advice
Apple.eat method invoked.
this is after around advice
```

​    执行结果中两次打印的create MyAspect instance表示当前切面实例创建了两次，这也符合我们进行的两次获取Apple实例。