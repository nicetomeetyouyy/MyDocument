#### 1.Mybatis简介
* MyBatis 是一款优秀的持久层框架，传统Java应用采用JDBC访问数据库，Mybatis通过使用映射配置文件，将SQL语句及结果返回到POJO（普通java对象），将SQL与程序代码分离

#### 2.Mybatis持久化操作步骤
* 开发持久化的PO类和编写持久化操作的Mapper.xml文件
* 获取SQLSessionFactory<br>

&emsp;&emsp;&emsp;SQLSessionFactory是Mybatis的关键对象，每一个Mybatis应用都以SQLSessionFactory对象为核心

```
 public SqlSession openSession()
 //返回SqlSession对象
```

* 获取SqlSession
<br>
&emsp;SqlSession 是执行持久化操作的对象，类似JDBC的connecttion，使用后应该关闭它


* 用面对对象方式操作数据库
* 关闭事务关闭SqlSession

#### 3.Mybatis配置文件结构
* configuration 最顶层
* proprerties 属性
  <br> &emsp; 可以在配置文件中配置proprerties属性,在此引入
* settings 设置
<br> &emsp;完整的配置示例如下）

```
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

* typeAliases类型别名
<br> &emsp;
类型别名是为 Java 类型设置一个短的名字

```
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
</typeAliases>
```
当这样配置,AUthor
可以在任何地方使用domain.blog.Author这个类<br> &emsp;
也可以指定包名

```
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

* typeHandlers类型处理器<br> &emsp;
MyBatis会用类型处理器将获取的值以合适的方式转换成 Java 类型。可以重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型，然后在这映射到一个JDBC

* objectFactory对象工厂<br> &emsp;
MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成
* plugins插件
* environments环境配置
   * environment环境变量
       * transactionManager（事务管理器）
       * dataSource数据源
<br> &emsp;
yBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中
```
<environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc"/>
                <property name="url" value=""/>
                <property name="username" value="root}"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
```

* databaseIdProvider数据库厂商标识
* mappers映射器
  定义去哪儿寻找sql映射文件

```
<mappers>
        <mapper resource="userMapper.xml"/>
    </mappers>
```

#### 4.XML映射文件
* cache – 对给定命名空间的缓存配置。
* cache-ref – 对其他命名空间缓存配置的引用。
* resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
* sql – 可被其他语句引用的可重用语句块。

```
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
```
被重用：

```
<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```

* insert – 映射插入语句

```
<insert id="insertUser" parameterType="com.xjm.mybatis.User" statementType="PREPARED">
        insert into users
    </insert>
```

* update – 映射更新语句

```
    <update id="updateUser">
        update users set
    </update>
```

* delete – 映射删除语句
  
```
  <delete id="deleteUser" >
        delete from users where id =#{id}
    </delete>
```

* select – 映射查询语句

```
<select id="selectBlog" parameterType="int" resultType="Blog">
    select * from users = #{id}
</select>
```
#{id}会告诉MyBatis 创建一个预处理语句（PreparedStatement）参数

#### 5.多表关联查询
* 一对一查询
  * 编写一个实体的扩展类添加SELECT返回数据所需要的数据，resulType就用这个扩展了接收
  * 使用ResultMap自定义类型
  
```
   <!-- =============== 一对一 =================-->
    <!--如果模型里有模型，使用resultMap-->
    <resultMap id="orderRslMap" type="orders">
        <!-- 往orders的模型匹配数据-->
        <id column="id" property="id"></id>
        <id column="note" property="note"></id>
        <id column="number" property="number"></id>
        <id column="createtime" property="createtime"></id>

        <!-- 往orders的user匹配数据
         模型里有模型，使用association来配置-->
        <association property="user" javaType="user">
            <id column="user_id" property="id"></id>
            <id column="username" property="username"></id>
            <id column="address" property="address"></id>
        </association>
    </resultMap>
```

* 一对多查询<br>
  在一的模型封装建立list类型的模型设置ResultTypeMap设置接收

```
<resultMap id="orderRslMap3" type="orders">
        <!-- 往orders的模型匹配数据-->
        <id column="id" property="id"></id>
        <id column="note" property="note"></id>
        <id column="number" property="number"></id>
        <id column="createtime" property="createtime"></id>
        <!-- 一对多匹配： 往orders的orderdetails 匹配数据
        注意：集合里类型使用ofType,而不javaType
        -->
        <collection property="orderDetails" ofType="orderDetail">
            <id column="detail_id" property="id"></id>
            <id column="items_id" property="itemsId"></id>
            <id column="items_num" property="itemsNum"></id>
        </collection>
    </resultMap>
```
* 多对多查询
<br>和一对多类似，conllection中可以嵌套显示

```
<collection property="orderList" ofType="orders">
            <id column="order_id" property="id"></id>
            <result column="number" property="number"/>
            <result column="createtime" property="createtime"/>
            <result column="note" property="note"/>

            <!-- 3.匹配Orders里有orderDetails-->
            <collection property="orderDetails" ofType="orderDetail">
                <id column="detail_id" property="id"></id>
                <result column="items_id" property="itemsId"/>
                <result column="items_num" property="itemsNum"/>

                <!-- 4.配置定单详情的商品信息-->
                <association property="items" javaType="items">
                    <id column="items_id" property="id"/>
                    <result column="name" property="name"/>
                    <result column="price" property="price"/>
                    <result column="detail" property="detail"/>
                </association>
            </collection>
        </collection>
```
#### 5.缓存
* 一级缓存
   * Mybatis对缓存提供支持，但是在没有配置的默认情况下，它只开启一级缓存，一级缓存只是相对于同一个SqlSession而言，使用同一个SqlSession对象调用一个Mapper方法，往往只执行一次SQL，因为使用SelSession第一次查询后，MyBatis会将其放在缓存中，以后再查询的时候，如果没有声明需要刷新，并且缓存没有超时的情况下，SqlSession都会取出当前缓存的数据，而不会再次发送SQL到数据库
   * MyBatis在开启一个数据库会话时，会 创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象。Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。
   * 如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用
   * 如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用
   * SqlSession中执行了任何一个update操作(update()、delete()、insert()) ，都会清空PerpetualCache对象的数据，但是该对象可以继续使用
   * mybatis认为，对于两次查询，如果以下条件都完全一样，那么就认为它们是完全相同的两次查询。
       * 传入的statementId
       * 查询时要求的结果集中的结果范围
       * 这次查询所产生的最终要传递给JDBC java.sql.Preparedstatement的Sql语句字符串（boundSql.getSql() ）
       * 传递给java.sql.Statement要设置的参数值
      
* 二级缓存
   * SqlSessionFactory层面上的二级缓存默认是不开启的，二级缓存的开席需要进行配置，实现二级缓存的时候，MyBatis要求返回的POJO必须是可序列化的。 也就是要求实现Serializable接口，配置方法很简单，只需要在映射XML文件配置就可以开启缓存了<cache/>
   映射语句文件中的所有select语句将会被缓存。
映射语句文件中的所欲insert、update和delete语句会刷新缓存。
缓存会使用默认的Least Recently Used（LRU，最近最少使用的）算法来收回。
根据时间表，比如No Flush Interval,（CNFI没有刷新间隔），缓存不会以任何时间顺序来刷新。
缓存会存储列表集合或对象(无论查询方法返回什么)的1024个引用
缓存会被视为是read/write(可读/可写)的缓存，意味着对象检索不是共享的，而且可以安全的被调用者修改，不干扰其他调用者或线程所做的潜在修改
 
