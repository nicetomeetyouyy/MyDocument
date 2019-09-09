#### 1.ApplicationContext的三个常用实现类
* ClassPathXmlApplicationContext：<br>
 在类加载路径下的配置文件，要求配置文件必须在类路径下

```
ApplicationContext ac =new ClassPathXmlApplicationContext（"xml配置文件"）

```

* FileSystemXmlApplicationContext：<br>
 加载磁盘下任意配置文件（有访问权限）

```
ApplicationContext ac =new ClassPathXmlApplicationContext（"路径\\xml配置文件"）
```

* AnnotationConfigApplicationContext:<br>
  读取注解创建容器
#### 2.两个容器接口区别
* ApplicationCont在构建容器时，采用立即加载方式，读取完配置文件马上创建配置文件中的对象，适用**单例模式**，但此接口可根据对象决定是否立即加载<br>
   * **bean 配置scpoe属性指定bean的作用范围**
       * singleton:单例
       * prototype:多例
       * request：web应用的请求范围
       * session：web应用的会话范围
       * global-session：集群环境的会话范围，不是集群环境，为session的会话范围
* BeanFactory采用延迟加载的方式，对象需要使用才加载，适用**多例模式**

#### 3.创建bean的三种方式
3.1 使用默认构造函数

```
<bean
id="demoService" class="com.xjm.service.demoServiceImpl">
</bean>
```
3.2 使用普通工厂（其他类）创建bean

```
//先加载普通类
<bean id ="demoFactory" class="com.xjm.demoFactory"></bean>
<bean id ="demoService" factory-bean="demoFactory" factory-method="getdemoService"></bean>
```
3.3 使用普通工厂（其他类）静态方法

```
<bean id="demoFactory" class="com.xjm.staticFactory"factory-method="getdemoService"></bean> 
```
#### 4.Spring中的依赖注入
* Spring中依赖关系由Spring来维护，依赖注入可以理解为依赖关系的维护<br>
4.1 通过构造函数注入
     * 使用<constructor-arg>标签
        * index: 指定参数在构造函数参数列表的索引位置
        * type: 指定参数在构造函数中的数据类型
        * name: 指定参数在构造函数中的变量名,最常用的属性
        * value: 给基本数据类型和String类型赋值
        * ref: 给其它Bean类型的字段赋值,ref属性的值应为配置文件中配置的Bean的id

```
public class People{
    private String name;
    private Integer age;
    private Date birthday;
}
public People(String name, Integer age, Date birthday) {
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }
    public void setName(String name) {
		this.name = name;
	}
	public void setAge(Integer age) {
		this.age = age;
	}
	public void setBirthday(Date birthday) {
		this.birthday = birthday;
	}

--------------
//注入
<bean id="now" class="java.util.Date" scope="prototype"></bean>

<bean id="People" class="cn.xjm.People">
	<constructor-arg name="name" value="myname"></constructor-arg>
	<constructor-arg name="age" value="18"></constructor-arg>
	<!-- birthday字段为已经注册的bean对象,其id为now -->
	<constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>

```
4.2 使用set方法注入

```
<!-- 使用Date类的无参构造函数创建Date对象 -->
<bean id="now" class="java.util.Date" scope="prototype"></bean>

<bean id="accountService" class="cn.maoritian.service.impl.AccountServiceImpl">
	<property name="name" value="myname"></property>
	<property name="age" value="21"></property>
	<!-- birthday字段为已经注册的bean对象,其id为now -->
	<property name="birthday" ref="now"></property>
</bean>

```
4.3集合对象的注入
，采用set方法
 * 数组字段: <array>标签表示集合,<value>标签表示集合内的成员
 * List字段: <list>标签表示集合,<value>标签表示集合内的成员.
 * Set字段: <set>标签表示集合,<value>标签表示集合内的成员
 * Map字段: <map>标签表示集合,<entry>标签表示集合内的键值对,其key属性表示键,value属性表示值
 * Properties字段: <props>标签表示集合,<prop>标签表示键值对,其key属性表示键,标签内的内容表示值.
