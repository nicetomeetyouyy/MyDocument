#### 1.三层架构
* 表现层：  也就是我们常说的web层。它负责接收客户端请求，向客户端响应结果，通常客户端使用http协议请求 web 层，web 需要接收 http 请求，完成 http 响应。  表现层包括**展示层和控制层**：**控制层负责接收请求，展示层负责结果的展示**。   表现层的设计一般都使用 MVC 模型。（MVC 是表现层的设计模型，和其他层没有关系） 
* 业务层：  也就是我们常说的 service 层。它**负责业务逻辑处理**，web 层依赖业 务层，但是业务层不依赖 web 层。  业务层在业务处理时可能会依赖持久层。
* 持久层：  **负责数据持久化**，持久层就是和数据库交互。
#### 2. MVC模型
*  MVC全名是Model View Controller 模型视图控制器，每个部分各司其职。 
*  Model：数据模型，JavaBean的类，用来进行数据封装。 
*   View：指JSP、HTML用来展示数据给用户
*   Controller：用来接收用户的请求，整个流程的控制器。用来进行数据校验等。 

#### 3.Spring环境配置
3.1  Maven坐标导入相关jar包
3.2 配置核心控制器和xml配置文件导入
* dispatcherServlet 是整个流程控制的中心，由 它调用其它组件处理用户的请求，dispatcherServlet 的存在降低了组件之间的耦合性
```
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```
3.3 在Springmvc配置文件配置视图解析器
* 视图解析器负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名 即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户

```
<bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/views/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>
```
3.4 Springmvc配置文件配置配置 <mvc:annotation-driven>
*  SpringMVC中还有HandlerMapping：处理器映射器 和 HandlAdapter：处理器适配器 两大组件
*  使用<mvc:annotation-driven> 自动加载这两大组件
```
<mvc:annotation-driven></mvc:annotation-driven>
```
#### 4. RequestMapping注解
4.1 RequestMapping的属性
* value：用于指定请求的 URL。它和 path 属性的作用是一样的.如下默认为value
```
@RequestMapping("melist")
```
* method：用于指定请求的方式。
* params：用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的 key 和 value 必须和 配置的一模一样。

*  headers ：发送的请求中必须包含的请求头

4.2 RequestMapping注解可以作用在方法和类上
*  作用在类上：第一级的访问目录 
*  作用在方法上：第二级的访问目录

#### 5.请求参数的绑定
5.1 表单提交name和参数名称一致（区分大小写）

```
<a href="account/findAccount?accountId=10&accountName=zhangsan">

@RequestMapping("/findAccount") 
public String findAccount(Integer accountId,String accountName) 
```
5.2 表单提交name和pojo类中的属性参数一致可直接使用pojo类型接受数据

```
@RequestMapping("/saveUser") 
public String saveAccount(User user)
```
5.3 如果一个pojo类中包含其他的引用类型，那么表单的name属性需要编写成：对象.属性 例如： address.name 
5.4 pojo类中包含集合属性的话，表单的name属性要编写成 集合名[?].属性，例如：

```
private List<Account> accounts; 
//表单描写：
<input type="text" name="accounts[0].name" ><br/>  
<input type="text" name="accounts[0].money" ><br/> 

```
#### 6.响应数据
6.1 String<br>
返回字符串可以指定逻辑视图的名称，根据视图解析器为物理视图的地址，如下返回successs.jsp页面
```
@RequestMapping("/hello")
    public String hello(){
        System.out.println("hello");
        return "successs";
    }
```
6.2 void 
* 系统自动返回当前路径名的同名JSP相关文件
* 可以在里面作重定向或者请求转发,地址不要带有相应的jsp结尾，会找不到对应文件，可以转发有控制器处理的地址
   
```
//重定向
    @RequestMapping("/returnVoid")
    public void returnVoid(HttpServletRequest request,HttpServletResponse response) throws IOException {
      response.sendRedirect("hello");
//将地址转发到hello，以上控制函数再跳转到相对页面
request.getRequestDispatcher("hello").forward(request,response);
    }
```
* 返回值是ModelAndView对象

```
public ModelAndView returnMAV(){
        ModelAndView mv=new ModelAndView();
        mv.addObject("user","haha");
        mv.addObject("age",1);
        mv.setViewName("successs");
        return mv;
    }
    //前端获取相对应的值
    ${requestScope.user}
  ${requestScope.age}
```
* 在控制器中请求转发的另一种写法:
转发写JSP文件路径可以打开网页，重定向不行，要写地址，根据控制器去转发页面
```
 @RequestMapping("/hello")
    public String hello(){
        System.out.println("hello");
//        return "successs";
//        正确写法
//    return "forward:WEB-INF/view/successs.jsp";
//  错误写法 return "forward:WEB-INF/view/successs";
   return "redirect:returnMAV";
        }
```

