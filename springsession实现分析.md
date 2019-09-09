### 核心模块

- SessionRepositoryFilter：Servlet规范中Filter的实现，用来切换HttpSession至Spring Session，包装HttpServletRequest和HttpServletResponse
- HttpServerletRequest/HttpServletResponse/HttpSessionWrapper包装器：包装原有的HttpServletRequest、HttpServletResponse和SpringSession，实现切换Session和透明继承HttpSession的关键之所在
- Session：Spring Session模块
- SessionRepository：管理Spring Session的模块
- HttpSessionStrategy：映射HttpRequst和HttpResponse到Session的策略

![1567561346312](D:%5Cydyun%5Cpicture%5C1567561346312.png)

### 1.SessionRepositoryFilter

SessionRepositoryFilter是一个Filter过滤器，符合Servlet的规范定义，用来修改包装请求和响应。这里负责包装切换HttpSession至Spring Session的请求和响应

```java
protected void doFilterInternal(HttpServletRequest request,
        HttpServletResponse response, FilterChain filterChain)
                throws ServletException, IOException {
    // 设置SessionRepository至Request的属性中
    request.setAttribute(SESSION_REPOSITORY_ATTR, this.sessionRepository);
    // 包装原始HttpServletRequest至SessionRepositoryRequestWrapper
    SessionRepositoryRequestWrapper wrappedRequest = new SessionRepositoryRequestWrapper(
            request, response, this.servletContext);
    // 包装原始HttpServletResponse响应至SessionRepositoryResponseWrapper
    SessionRepositoryResponseWrapper wrappedResponse = new SessionRepositoryResponseWrapper(
            wrappedRequest, response);
    // 设置当前请求的HttpSessionStrategy策略
    HttpServletRequest strategyRequest = this.httpSessionStrategy
            .wrapRequest(wrappedRequest, wrappedResponse);
    // 设置当前响应的HttpSessionStrategy策略
    HttpServletResponse strategyResponse = this.httpSessionStrategy
            .wrapResponse(wrappedRequest, wrappedResponse);
    try {
        filterChain.doFilter(strategyRequest, strategyResponse);
    }
    finally {
        // 提交session
        wrappedRequest.commitSession();
    }
}
```

以上是SessionRepositoryFilter的核心操作，每个HttpRequest进入，都会被该Filter包装成切换Session的请求和响应对象

### 2.SessionRepositoryRequestWrapper

在spring session中request的实际类型SessionRepositoryRequestWrapper。调用SessionRepositoryRequestWrapper的getSession方法会触发创建spring session，而非web容器的HttpSession.

```
private final class SessionRepositoryRequestWrapper
            extends HttpServletRequestWrapper {

    private final String CURRENT_SESSION_ATTR = HttpServletRequestWrapper.class
                .getName();

    // 当前请求sessionId有效
    private Boolean requestedSessionIdValid;
    // 当前请求sessionId无效
    private boolean requestedSessionInvalidated;
    private final HttpServletResponse response;
    private final ServletContext servletContext;

    private SessionRepositoryRequestWrapper(HttpServletRequest request,
            HttpServletResponse response, ServletContext servletContext) {
        // 调用HttpServletRequestWrapper构造方法，实现包装
        super(request);
        this.response = response;
        this.servletContext = servletContext;
    }
}
```

SessionRepositoryRequestWrapper继承Servlet规范中定义的包装器HttpServletRequestWrapper HttpServletRequestWrapper是Servlet规范api提供的用于扩展HttpServletRequest的扩张点——即装饰器模式，可以通过重写一些api达到功能点的增强和自定义. SessionRepositoryRequestWrapper继承HttpServletRequestWrapper，在构造方法中将原有的HttpServletRequest通过调用super完成对HttpServletRequestWrapper中持有的HttpServletRequest初始化赋值，然后重写和session相关的方法。这样就保证SessionRepositoryRequestWrapper的其他方法调用都是使用原有的HttpServletRequest的数据，只有session相关的是重写的逻辑

```java
@Override
public HttpSessionWrapper getSession(boolean create) {
    // 从当前请求的attribute中获取session，如果有直接返回
    HttpSessionWrapper currentSession = getCurrentSession();
    if (currentSession != null) {
        return currentSession;
    }

    // 获取当前request的sessionId，这里使用了HttpSessionStrategy
    // 决定怎样将Request映射至Session，默认使用Cookie策略，即从cookies中解析sessionId
    String requestedSessionId = getRequestedSessionId();
    // 请求的如果sessionId存在且当前request的attribute中的没有session失效属性
    // 则根据sessionId获取spring session
    if (requestedSessionId != null
            && getAttribute(INVALID_SESSION_ID_ATTR) == null) {
        S session = getSession(requestedSessionId);
        // 如果spring session不为空，则将spring session包装成HttpSession并
        // 设置到当前Request的attribute中，防止同一个request getsession时频繁的到存储器
        //中获取session，提高性能
        if (session != null) {
            this.requestedSessionIdValid = true;
            currentSession = new HttpSessionWrapper(session, getServletContext());
            currentSession.setNew(false);
            setCurrentSession(currentSession);
            return currentSession;
        }
        // 如果根据sessionId，没有获取到session，则设置当前request属性，此sessionId无效
        // 同一个请求中获取session，直接返回无效
        else {
            // This is an invalid session id. No need to ask again if
            // request.getSession is invoked for the duration of this request
            if (SESSION_LOGGER.isDebugEnabled()) {
                SESSION_LOGGER.debug(
                        "No session found by id: Caching result for getSession(false) for this HttpServletRequest.");
            }
            setAttribute(INVALID_SESSION_ID_ATTR, "true");
        }
    }
    // 判断是否创建session
    if (!create) {
        return null;
    }
    if (SESSION_LOGGER.isDebugEnabled()) {
        SESSION_LOGGER.debug(
                "A new session was created. To help you troubleshoot where the session was created we provided a StackTrace (this is not an error). You can prevent this from appearing by disabling DEBUG logging for "
                        + SESSION_LOGGER_NAME,
                new RuntimeException(
                        "For debugging purposes only (not an error)"));
    }
    // 根据sessionRepository创建spring session
    S session = SessionRepositoryFilter.this.sessionRepository.createSession();
    // 设置session的最新访问时间
    session.setLastAccessedTime(System.currentTimeMillis());
    // 包装成HttpSession透明化集成
    currentSession = new HttpSessionWrapper(session, getServletContext());
    // 设置session至Requset的attribute中，提高同一个request访问session的性能
    setCurrentSession(currentSession);
    return currentSession;
}
```

上述SessionRepositoryFilter在包装HttpServletRequest后，执行FilterChain中使用finally保证请求的Session始终session会被提交，此提交操作中将sesionId设置到response的head中并将session持久化至存储器中。

持久化只持久spring session，并不是将spring session包装后的HttpSession持久化，因为HttpSession不过是包装器，持久化没有意义

```java
/**
 * Uses the HttpSessionStrategy to write the session id to the response and
 * persist the Session.
 */
private void commitSession() {
    // 获取当前session
    HttpSessionWrapper wrappedSession = getCurrentSession();
    // 如果当前session为空，则删除cookie中的相应的sessionId
    if (wrappedSession == null) {
        if (isInvalidateClientSession()) {
            SessionRepositoryFilter.this.httpSessionStrategy
                    .onInvalidateSession(this, this.response);
        }
    }
    else {
        // 从HttpSession中获取当前spring session
        S session = wrappedSession.getSession();
        // 持久化spring session至存储器
        SessionRepositoryFilter.this.sessionRepository.save(session);
        // 如果是新创建spring session，sessionId到response的cookie
        if (!isRequestedSessionIdValid()
                || !session.getId().equals(getRequestedSessionId())) {
            SessionRepositoryFilter.this.httpSessionStrategy.onNewSession(session,
                    this, this.response);
        }
    }
}
```

### 3.SessionRepositoryResponseWrapper

```java
 private final class SessionRepositoryResponseWrapper
        extends OnCommittedResponseWrapper {
    private final SessionRepositoryRequestWrapper request;
    /**
     * Create a new {@link SessionRepositoryResponseWrapper}.
     * @param request the request to be wrapped
     * @param response the response to be wrapped
     */
    SessionRepositoryResponseWrapper(SessionRepositoryRequestWrapper request,
            HttpServletResponse response) {
        super(response);
        if (request == null) {
            throw new IllegalArgumentException("request cannot be null");
        }
        this.request = request;
    }
    @Override
    protected void onResponseCommitted() {
        this.request.commitSession();
    }
}
```

### 4. Session

![1567560113241](D:%5Cydyun%5Cpicture%5C1567560113241.png)

* Session是spring-session对session的抽象，主要是为了鉴定用户，为Http请求和响应提供上下文过程，该Session可以被HttpSession、WebSocket Session，非WebSession等使用。定义了Session的基本行为：
  * getId：获取sessionId
  * setAttribute：设置session属性
  * getAttribte：获取session属性
* ExipringSession：提供Session额外的过期特性。定义了以下关于过期的行为：
  * setLastAccessedTime：设置最近Session会话过程中最近的访问时间
  * getLastAccessedTime：获取最近的访问时间
  * setMaxInactiveIntervalInSeconds：设置Session的最大闲置时间
  * getMaxInactiveIntervalInSeconds：获取最大闲置时间
  * isExpired：判断Session是否过期
* MapSession：基于java.util.Map的ExpiringSession的实现

```java
public final class MapSession implements ExpiringSession, Serializable {
    /**
     * Default {@link #setMaxInactiveIntervalInSeconds(int)} (30 minutes).
     */
    public static final int DEFAULT_MAX_INACTIVE_INTERVAL_SECONDS = 1800;

    private String id;
    private Map<String, Object> sessionAttrs = new HashMap<String, Object>();
    private long creationTime = System.currentTimeMillis();
    private long lastAccessedTime = this.creationTime;

    /**
     * Defaults to 30 minutes.
     */
    private int maxInactiveInterval = DEFAULT_MAX_INACTIVE_INTERVAL_SECONDS;
```



* RedisSession：基于MapSession和Redis的ExpiringSession实现，提供Session的持久化能力

  ```java
  /**
   * A custom implementation of {@link Session} that uses a {@link MapSession} as the
   * basis for its mapping. It keeps track of any attributes that have changed. When
   * {@link org.springframework.session.data.redis.RedisOperationsSessionRepository.RedisSession#saveDelta()}
   * is invoked all the attributes that have been changed will be persisted.
   *
   * @author Rob Winch
   * @since 1.0
   */
  final class RedisSession implements ExpiringSession {
      private final MapSession cached;
      private Long originalLastAccessTime;
      private Map<String, Object> delta = new HashMap<String, Object>();
      private boolean isNew;
      private String originalPrincipalName;
  ```

  
  * cached：实际上是一个MapSession实例，用于做本地缓存，每次在getAttribute时无需从Redis中获取，主要为了improve性能
  * delta：用于跟踪变化数据，做持久化
  
  函数：
  
  
  * RedisSession中最为重要的行为saveDelta——持久化Session至Redis中：
  
    * 保存Session中的属性数据至Redis中
    * 清空delta中数据，防止重复提交Session中的数据
    * 更新过期时间至下一个过期时间间隔的时刻
  * setMaxInactiveIntervalInSeconds： 设置session的存活时间，即最大过期时间。先保存至本地缓存，然后再保存至delta
  * getMaxInactiveIntervalInSeconds()：直接从本地缓存获取过期时间
  * getAttribute：直接从本地缓存中获取Session中的属性
  * setAttribute：保存Session属性至本地缓存和delta中
  
### 5.SessionRepository

SessionRepository为管理spring-session的接口实例：

```java
S createSession();
void save(S session);
S getSession(String id);
void delete(String id);
```

  

### 6.HttpSessionStrategy

HttpSessionStrategy是建立Request/Response和Session之间的映射关系的策略。

该策略接口中定义一套策略行为：

```java
// 根据请求获取SessionId，即建立请求至Session的映射关系
String getRequestedSessionId(HttpServletRequest request);
// 对于新创建的Session，通知客户端
void onNewSession(Session session, HttpServletRequest request,
            HttpServletResponse response);
// 对于session无效，通知客户端
void onInvalidateSession(HttpServletRequest request, HttpServletResponse response);
```



​    

​    

 

