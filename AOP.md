#### 1.SpringAOP相关术语
* AOP为Aspect Oriented Programming的缩写，意为：**面向切面编程**，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

* **target(目标类)**：需要被代理的类。例如：UserService

* **Joinpoint(连接点)**: 被拦截到的方法.

* **Pointcut(切入点)**: 我们对其进行增强的方法.

* **Advice(通知/增强)**: 对切入点进行的增强操作

包括**前置通知,后置通知,异常通知,最终通知,环绕通知**
*  proxy(代理类)

* **Weaving(织入)**: 是指把增强应用到目标对象来创建新的代理对象的过程。

* **Aspect(切面)**: 是切入点和通知的结合

#### 2.XML配置

2.1 在XML中引入aop相关依赖并引入相关bean

```
xmlns:aop="http://www.springframework.org/schema/aop"
http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop.xsd"
<!-- 2 创建切面类（通知） --> 
<bean id="logger" class="com.logger"></bean>
```
2.2 AOP配置
* <aop:config>标签声明AOP配置
* <aop:aspect>标签配置切面,id指定切面的id
,ref: 引用通知类的id
* <aop:pointcut>标签配置切入点表达式,**expression**: 指定切入点表达式
* <aop:xxx>标签配置对应类型的通知方法
  * **<aop:before>:** 配置前置通知,指定的增强方法在切入点方法之前执行.
  * **<aop:after-returning>:** 配置后置通知,指定的增强方法在切入点方法正常执行之后执行.
  * **<aop:after-throwing>:** 配置异常通知,指定的增强方法在切入点方法产生异常后执行.
  * **<aop:after>:** 配置最终通知,无论切入点方法执行时是否发生异常,指定的增强方法都会最后执行.
  * **<aop:around>:** 配置环绕通知,可以在代码中手动控制增强代码的执行时机
  * **method:** 指定通知类中的增强方法名.
**ponitcut-ref:** 指定切入点的表达式的id

```
<aop:config>
    <aop:aspect id="logAdvice" ref="logger">
        <!--指定切入点表达式-->
        <aop:pointcut expression="execution(* cn,maoritian.service.impl.*.*(..))" id="pt1"></aop:pointcut>
        <!--配置各种类型的通知-->
        <aop:before method="printLogBefore" pointcut-ref="pt1"></aop:before>
        <aop:after-returning method="printLogAfterReturning" pointcut-ref="pt1"></aop:after-returning>
        <aop:after-throwing method="printLogAfterThrowing" pointcut-ref="pt1"></aop:after-throwing>
        <aop:after method="printLogAfter" pointcut-ref="pt1"></aop:after>
		<!--环绕通知一般单独使用-->       
        <!-- <aop:around method="printLogAround" pointcut-ref="pt1"></aop:around> -->
    </aop:aspect>
</aop:config>
```
#### 3.切入点表达式
切入点表达式：execution([修饰符] 返回值类型 包路径.类名.方法名(参数))<br>
 例如：
```
<aop:pointcut 
expression="execution(public void cn.service.impl.AccountServiceImpl.saveAccount(cn.Account))" id="pt1">
</aop:pointcut>
```
访问修饰符可省略：

```
<aop:pointcut 
expression="execution( void cn.service.impl.AccountServiceImpl.saveAccount(cn.Account))" id="pt1">
</aop:pointcut>
```
可以用*表示任意返回值

```
<aop:pointcut 
expression="execution( * cn.service.impl.AccountServiceImpl.saveAccount(cn.Account))" id="pt1">
</aop:pointcut>
```
可以包路径可以使用*.表示任意包. 但是*.的个数要和包的层级数相匹配:

```
<aop:pointcut 
expression="execution( * *.*.*.AccountServiceImpl.saveAccount(cn.Account))" id="pt1">
</aop:pointcut>
```
包路径可以使用*..,表示当前包,及其子包,xml放在根路径下可以匹配所有包：

```
<aop:pointcut 
expression="execution( * *..AccountServiceImpl.saveAccount(cn.Account))" id="pt1">
</aop:pointcut>
```
类名和方法名也可以用*表示任意类或者任意方法：

```
<aop:pointcut 
expression="execution( * *..*.saveAccount(cn.Account))" id="pt1">
</aop:pointcut>
<aop:pointcut 
expression="execution( * *..*.*(cn.Account))" id="pt1">
</aop:pointcut>
```
参数名可以用*表示任意参数但必须有参数，..可以表示任意参数包括无参

```
<aop:pointcut 
expression="execution( * *..AccountServiceImpl.saveAccount(*))" id="pt1">
</aop:pointcut>
<aop:pointcut 
expression="execution( * *..AccountServiceImpl.saveAccount(..))" id="pt1">
</aop:pointcut>
```
#### 4.注解使用AOP
Srping需要在配置文件声明对AOP的注解支持
```
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```
Springboot无需配置

* **@Aspect:** 声明当前类为通知类
* **@Before:** 声明该方法为前置通知.相当于xml配置中的<aop:before>标签
* **@AfterReturning:** 声明该方法为后置通知.相当于xml配置中的<aop:after-returning>标签
* **@AfterThrowing:** 声明该方法为异常通知.相当于xml配置中的<aop:after-throwing>标签
* **@After:** 声明该方法为最终通知.相当于xml配置中的<aop:after>标签
* **@Around:** 声明该方法为环绕通知.相当于xml配置中的<aop:around>标签
**@Pointcut:**指定切入点表达式,如下声明一个空方法:

```
@Pointcut("execution(* cn.service.impl.*.*(..))")
    private void pt1(){} 
   // 通过调用被注解的方法获取切入点表达式
    @AfterReturning("pt1()")
    public void printLogAfterReturning(){
        System.out.println("后置通知Logger类中的printLogAfterReturning方法开始记录日志了。。。");
    }

```



