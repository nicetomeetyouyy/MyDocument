#### 1.Spring事务控制
* JavaEE 体系进行分层开发,事务处理位于业务层,Spring提供了分层设计业务层的事务处理解决方案
* Spring 框架为我们提供了一组事务控制的接口,这组接口在spring-tx-5.0.2.RELEASE.jar中
* Spring 的事务控制都是基于AOP的,它既可以使用配置的方式实现,也可以使用编程的方式实现.推荐使用配置方式实现
#### 2.事务控制API
2.1 **PlatformTransactionManager**接口是Spring提供的事务管理器,它提供了操作事务的方法如下:
   * TransactionStatus getTransaction(TransactionDefinition definition): 获得事务状态信息
   * void commit(TransactionStatus status): 提交事务
   * void rollback(TransactionStatus status): 回滚事务
 <br> <br> 
其实现类：
    * **DataSourceTransactionManager**使用SpringJDBC或iBatis进行持久化数据时使用
    * **HibernateTransactionManager**使用Hibernate版本进行持久化数据时使用 

2.2 **TransactionDefinition**: 事务定义信息对象,提供查询事务定义的方法如下：
* String getName(): 获取事务对象名称
* int getIsolationLevel(): 获取事务隔离级别,设置两个事务之间的数据可见性
* 事务的隔离级别：
   * **ISOLATION_DEFAULT:** Spring事务管理的的默认级别,使用数据库默认的事务隔离级别.
   * **ISOLATION_READ_UNCOMMITTED:** 读未提交.
   * **ISOLATION_READ_COMMITTED:** 读已提交.
   * **ISOLATION_REPEATABLE_READ:** 可重复读.
   * **ISOLATION_SERIALIZABLE:** 串行化.
 
2.3 **getPropagationBehavior():** 获取**事务传播行为**,设置新事务是否事务以及是否使用当前事务。 通常使用的是前两种: **REQUIRED**和**SUPPORTS**.事务传播行为如下:
* **REQUIRED**: Spring默认事务传播行为. 若当前没有事务,就新建一个事务;若当前已经存在一个事务中,加入到这个事务中.增删改查操作均可用
* **SUPPORTS**: 若当前没有事务,就不使用事务;若当前已经存在一个事务中,加入到这个事务中.查询操作可用
* **MANDATORY**: 使用当前的事务,若当前没有事务,就抛出异常
* **REQUERS_NEW**: 新建事务,若当前在事务中,把当前事务挂起
* **NOT_SUPPORTED**: 以非事务方式执行操作,若当前存在事务,就把当前事务挂起
* **NEVER**:以非事务方式运行,若当前存在事务,抛出异常
* **NESTED**:若当前存在事务,则在嵌套事务内执行;若当前没有事务,则执行REQUIRED类似的操作

2.4 **int getTimeout()**: 获取事务超时时间. Spring默认设置事务的超时时间为-1,表示永不超时

2.5 **boolean isReadOnly()**: 获取事务是否只读. Spring默认设置为false,建议查询操作中设置为true

2.6 **TransactionStatus**: 事务状态信息对象,提供操作事务状态的方法如下:
 * void flush(): 刷新事务
 * boolean hasSavepoint(): 查询是否存在存储点
 * boolean isCompleted(): 查询事务是否完成
 * boolean isNewTransaction(): 查询是否是新事务
 * boolean isRollbackOnly(): 查询事务是否回滚
 * void setRollbackOnly(): 设置事务回滚
 * 
 ### 3.Spring事务实现
3.1 准备事务

 * 收集@Transactional注解属性信息，生成事务定义对象
 * @Transactional 注解定义在不同位置的 优先级
为 ，实列方法，实列类，接口方法，接口类。不会取并集 ，也不会覆盖，按照优先级查找，直到找到为止
* 获取事务管理器
如果使用@Transactional 指定了使用哪个事务管理器 ，就会获取响应的事务管理器。如果没有 就从IOC 容器中获取

3.2 开启事务

收集到了事务定义信息，和事务管理器之后，就可以 利用PlatformTransactionManager.getTransactional 开启事务
* 事务挂起

  当线程中已经存在事务,在某些事务传播行为下就需要挂起外层事务，如PROPAGATION_NOT_SUPPORTED：不能运行在一个事务中，如果存在事务就挂起当前事务，执行
* 事务恢复
 
如果内部事务出现异常或者 内部事务提交 都会触发外层事务的恢复

3.3 事务回滚

如果事务运行过程中出现某些异常 会导致事务回滚

3.4 事务提交

只有当事务是一个 新事务的时候才会进行提交，mybatis 会在beforeCommit 中 执行Sqlsession commit