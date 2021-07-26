title: SpringSession源码（二）
author: wen
tags:
  - springsession
categories:
  - springsession
date: 2019-11-22 15:25:00
---
[如何使用spring session](https://smallwenzi.github.io/2019/11/13/ring-session%EF%BC%881%EF%BC%89/)
上一篇文章中介绍了Spring-Session的核心原理，Filter，Session，Repository等等，传送门：[SpringSession源码](https://smallwenzi.github.io/2019/11/21/SpringSession%E6%BA%90%E7%A0%81/)

这篇继上一篇的原理逐渐深入Spring-Session中的事件机制原理的探索。众所周知，Servlet规范中有对HttpSession的事件的处理，如：HttpSessionEvent/HttpSessionIdListener/HttpSessionListener，可以查看Package javax.servlet

在Spring-Session中也有相应的Session事件机制实现，包括Session创建/过期/删除事件。
本文主要从以下方面探索Spring-Session中事件机制

* Session事件的抽象
* 事件的触发机制
# Session事件的抽象
先来看下Session事件抽象UML类图，整体掌握事件之间的依赖关系。
![Session事件的抽象](pringSession源码（二）/1.png)
Session Event最顶层是ApplicationEvent，即Spring上下文事件对象。由此可以看出Spring-Session的事件机制是基于Spring上下文事件实现。

抽象的AbstractSessionEvent事件对象提供了获取Session（这里的是指Spring Session的对象）和SessionId。   

基于事件的类型，分类为：
* Session创建事件
* Session删除事件
* Session过期事件
事件对象只是对事件本身的抽象，描述事件的属性，如：

获取事件产生的源：getSource获取事件产生源
获取相应事件特性：getSession/getSessoinId获取时间关联的Session
下面再深入探索以上的Session事件是如何触发，从事件源到事件监听器的链路分析事件流转过程。
# 二.事件的触发机制


介绍Session Event事件基于Spring的ApplicationEvent实现。先简单认识spring上下文事件机制：
![Session事件的抽象](pringSession源码（二）/2.png)

* ApplicationEventPublisher实现用于发布Spring上下文事件ApplicationEvent
* ApplicationListener实现用于监听Spring上下文事件ApplicationEvent
* ApplicationEvent抽象上下文事件

那么在Spring-Session中必然包含事件发布者ApplicationEventPublisher发布Session事件和ApplicationListener监听Session事件。

可以看出ApplicationEventPublisher发布一个事件：
```
@FunctionalInterface
public interface ApplicationEventPublisher {

    /**
     * Notify all <strong>matching</strong> listeners registered with this
     * application of an application event. Events may be framework events
     * (such as RequestHandledEvent) or application-specific events.
     * @param event the event to publish
     * @see org.springframework.web.context.support.RequestHandledEvent
     */
    default void publishEvent(ApplicationEvent event) {
        publishEvent((Object) event);
    }

    /**
     * Notify all <strong>matching</strong> listeners registered with this
     * application of an event.
     * <p>If the specified {@code event} is not an {@link ApplicationEvent},
     * it is wrapped in a {@link PayloadApplicationEvent}.
     * @param event the event to publish
     * @since 4.2
     * @see PayloadApplicationEvent
     */
    void publishEvent(Object event);

}
```
ApplicationListener用于监听相应的事件：
```
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {

    /**
     * Handle an application event.
     * @param event the event to respond to
     */
    void onApplicationEvent(E event);

}
```
Session事件的流程实现如下：
![Session事件的抽象](pringSession源码（二）/3.png)
RedisMessageListenerContainer初始化时会启动一个线程SubscriptionTask进行订阅redis信息

上图展示了Spring-Session事件流程图，事件源来自于Redis键空间通知，在spring-data-redis项目中抽象MessageListener监听Redis事件源，然后将其传播至spring应用上下文发布者，由发布者发布事件。在spring上下文中的监听器Listener即可监听到Session事件。

因为两者是Spring框架提供的对Spring的ApplicationEvent的支持。Session Event基于ApplicationEvent实现，必然也有其相应发布者和监听器的的实现。

Spring-Session中的RedisSession的SessionRepository是RedisOperationSessionRepository。所有关于RedisSession的管理操作都是由其实现，所以Session的产生源是RedisOperationSessionRepository。

在RedisOperationSessionRepository中持有ApplicationEventPublisher对象用于发布Session事件。
```
private ApplicationEventPublisher eventPublisher = new ApplicationEventPublisher() {
    @Override
    public void publishEvent(ApplicationEvent event) {
    }
    @Override
    public void publishEvent(Object event) {
    }
};
```
但是该ApplicationEventPublisher是空实现，实际实现是在应用启动时由Spring-Session自动配置。在spring-session-data-redis模块中RedisHttpSessionConfiguration中有关于创建RedisOperationSessionRepository Bean时将调用set方法将ApplicationEventPublisher配置。
```
@Configuration
@EnableScheduling
public class RedisHttpSessionConfiguration extends SpringHttpSessionConfiguration
        implements BeanClassLoaderAware, EmbeddedValueResolverAware, ImportAware,
        SchedulingConfigurer {

    private ApplicationEventPublisher applicationEventPublisher;

    @Bean
    public RedisOperationsSessionRepository sessionRepository() {
        RedisTemplate<Object, Object> redisTemplate = createRedisTemplate();
        RedisOperationsSessionRepository sessionRepository = new RedisOperationsSessionRepository(
                redisTemplate);
        // 注入依赖
        sessionRepository.setApplicationEventPublisher(this.applicationEventPublisher);
        if (this.defaultRedisSerializer != null) {
            sessionRepository.setDefaultSerializer(this.defaultRedisSerializer);
        }
        sessionRepository
                .setDefaultMaxInactiveInterval(this.maxInactiveIntervalInSeconds);
        if (StringUtils.hasText(this.redisNamespace)) {
            sessionRepository.setRedisKeyNamespace(this.redisNamespace);
        }
        sessionRepository.setRedisFlushMode(this.redisFlushMode);
        return sessionRepository;
    }

    // 注入上下文中的ApplicationEventPublisher Bean
    @Autowired
    public void setApplicationEventPublisher(
            ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }

}
```
在进行自动配置时，将上下文中的ApplicationEventPublisher的注入，实际上即ApplicationContext对象。
对于ApplicationListener是由应用开发者自行实现，注册成Bean即可。当有Session Event发布时，即可监听。
```
/**
 * session事件监听器
 *
 * @author huaijin
 */
@Component
public class SessionEventListener implements ApplicationListener<SessionDeletedEvent> {

    private static final String CURRENT_USER = "currentUser";

    @Override
    public void onApplicationEvent(SessionDeletedEvent event) {
        Session session = event.getSession();
        UserVo userVo = session.getAttribute(CURRENT_USER);
        System.out.println("Current session's user:" + userVo.toString());
    }
}
```
以上部分探索了Session事件的发布者和监听者，但是核心事件的触发发布则是由Redis的键空间通知机制触发，当有Session创建/删除/过期时，Redis键空间会通知Spring-Session应用。

RedisOperationsSessionRepository实现spring-data-redis中的MessageListener接口。
```
/**
 * Listener of messages published in Redis.
 *
 * @author Costin Leau
 * @author Christoph Strobl
 */
public interface MessageListener {

    /**
     * Callback for processing received objects through Redis.
     *
     * @param message message must not be {@literal null}.
     * @param pattern pattern matching the channel (if specified) - can be {@literal null}.
     */
    void onMessage(Message message, @Nullable byte[] pattern);
}
```
该监听器即用来监听redis发布的消息。RedisOperationsSessionRepositorys实现了该Redis键空间消息通知监听器接口，实现如下：
```
public class RedisOperationsSessionRepository implements
        FindByIndexNameSessionRepository<RedisOperationsSessionRepository.RedisSession>,
        MessageListener {

    @Override
    @SuppressWarnings("unchecked")
    public void onMessage(Message message, byte[] pattern) {
        // 获取该消息发布的redis通道channel
        byte[] messageChannel = message.getChannel();
        // 获取消息体内容
        byte[] messageBody = message.getBody();

        String channel = new String(messageChannel);

        // 如果是由Session创建通道发布的消息，则是Session创建事件
        if (channel.startsWith(getSessionCreatedChannelPrefix())) {
            // 从消息体中载入Session
            Map<Object, Object> loaded = (Map<Object, Object>) this.defaultSerializer
                    .deserialize(message.getBody());
            // 发布创建事件
            handleCreated(loaded, channel);
            return;
        }

        // 如果消息体不是以过期键前缀，直接返回。因为spring-session在redis中的key命名规则：
        // "${namespace}:sessions:expires:${sessionId}"，如：
        // session.example:sessions:expires:a5236a19-7325-4783-b1f0-db9d4442db9a
        // 所以判断过期或者删除的键是否为spring-session的过期键。如果不是，可能是应用中其他的键的操作，所以直接return
        String body = new String(messageBody);
        if (!body.startsWith(getExpiredKeyPrefix())) {
            return;
        }

        // 根据channel判断键空间的事件类型del或者expire时间
        boolean isDeleted = channel.endsWith(":del");
        if (isDeleted || channel.endsWith(":expired")) {
            int beginIndex = body.lastIndexOf(":") + 1;
            int endIndex = body.length();
            // Redis键空间消息通知内容即操作的键，spring-session键中命名规则：
            // "${namespace}:sessions:expires:${sessionId}"，以下是根据规则解析sessionId
            String sessionId = body.substring(beginIndex, endIndex);

            // 根据sessionId加载session
            RedisSession session = getSession(sessionId, true);

            if (session == null) {
                logger.warn("Unable to publish SessionDestroyedEvent for session "
                        + sessionId);
                return;
            }

            if (logger.isDebugEnabled()) {
                logger.debug("Publishing SessionDestroyedEvent for session " + sessionId);
            }

            cleanupPrincipalIndex(session);

            // 发布Session delete事件
            if (isDeleted) {
                handleDeleted(session);
            }
            else {
                // 否则发布Session expire事件
                handleExpired(session);
            }
        }
    }
}
```
下续再深入每种事件产生的前世今生。

## Session创建事件的触发

![Session事件的抽象](pringSession源码（二）/4.png)
* 由RedisOperationSessionRepository向Redis指定通道${namespace}:event:created:${sessionId}发布一个message
* MessageListener的实现RedisOperationSessionRepository监听到Redis指定通道${namespace}:event:created:${sessionId}的消息
* 将其传播至ApplicationEventPublisher
* ApplicationEventPublisher发布SessionCreateEvent
* ApplicationListener监听SessionCreateEvent，执行相应逻辑

RedisOperationSessionRepository中保存一个Session时，判断Session是否新创建。
如果新创建，则向
```
@Override
public void save(RedisSession session) {
    session.saveDelta();
    // 判断是否为新创建的session
    if (session.isNew()) {
        // 获取redis指定的channel：${namespace}:event:created:${sessionId}，
        // 如：session.example:event:created:82sdd-4123-o244-ps123
        String sessionCreatedKey = getSessionCreatedChannel(session.getId());
        // 向该通道发布session数据
        this.sessionRedisOperations.convertAndSend(sessionCreatedKey, session.delta);
        // 设置session为非新创建
        session.setNew(false);
    }
}
```
该save方法的调用是由HttpServletResponse提交时——即返回客户端响应调用，上篇文章已经详解，这里不再赘述。关于RedisOperationSessionRepository实现MessageListener上述已经介绍，这里同样不再赘述。
## Session删除事件的触发
删除事件中使用到了Redis KeySpace Notification，建议先了解该技术。

![Session事件的抽象](pringSession源码（二）/5.png)
* 由RedisOperationSessionRepository删除Redis键空间中的指定Session的过期键，Redis键空间会向**__keyevent@*:del**的channel发布删除事件消息
* MessageListener的实现RedisOperationSessionRepository监听到Redis指定通道**__keyevent@*:del**的消息
* 将其传播至ApplicationEventPublisher
* ApplicationEventPublisher发布SessionDeleteEvent
* ApplicationListener监听SessionDeleteEvent，执行相应逻辑
当调用HttpSession的invalidate方法让Session失效时，即会调用RedisOperationSessionRepository的deleteById方法删除Session的过期键。
```
/**
 * Allows creating an HttpSession from a Session instance.
 *
 * @author Rob Winch
 * @since 1.0
 */
private final class HttpSessionWrapper extends HttpSessionAdapter<S> {
    HttpSessionWrapper(S session, ServletContext servletContext) {
        super(session, servletContext);
    }

    @Override
    public void invalidate() {
        super.invalidate();
        SessionRepositoryRequestWrapper.this.requestedSessionInvalidated = true;
        setCurrentSession(null);
        clearRequestedSessionCache();
        // 调用删除方法
        SessionRepositoryFilter.this.sessionRepository.deleteById(getId());
    }
}
```
上篇中介绍了包装Spring Session为HttpSession，这里不再赘述。这里重点分析deleteById内容：
```
@Override
public void deleteById(String sessionId) {
    // 如果session为空则返回
    RedisSession session = getSession(sessionId, true);
    if (session == null) {
        return;
    }

    cleanupPrincipalIndex(session);
    this.expirationPolicy.onDelete(session);
    // 获取session的过期键
    String expireKey = getExpiredKey(session.getId());
    // 删除过期键，redis键空间产生del事件消息，被MessageListener即
    // RedisOperationSessionRepository监听
    this.sessionRedisOperations.delete(expireKey);
    session.setMaxInactiveInterval(Duration.ZERO);
    save(session);
}
```
后续流程同SessionCreateEvent流程。

## Session失效事件的触发
Session的过期事件流程比较特殊，因为Redis的键空间通知的特殊性，Redis键空间通知不能保证过期键的通知的及时性。
![Session事件的抽象](pringSession源码（二）/6.png)
* RedisOperationsSessionRepository中有个定时任务方法每整分运行访问整分Session过期键集合中的过期sessionId，如：spring:session:expirations:1439245080000。触发Redis键空间会向**__keyevent@*:expired**的channel发布过期事件消息
* MessageListener的实现RedisOperationSessionRepository监听到Redis指定通道**__keyevent@*:expired**的消息
* 将其传播至ApplicationEventPublisher
* ApplicationEventPublisher发布SessionDeleteEvent
* ApplicationListener监听SessionDeleteEvent，执行相应逻辑

```
@Scheduled(cron = "0 * * * * *")
public void cleanupExpiredSessions() {
    this.expirationPolicy.cleanExpiredSessions();
}
```
定时任务每整分运行，执行cleanExpiredSessions方法。expirationPolicy是RedisSessionExpirationPolicy实例，是RedisSession过期策略。
```
public void cleanExpiredSessions() {
    // 获取当前时间戳
    long now = System.currentTimeMillis();
    // 时间滚动至整分，去掉秒和毫秒部分
    long prevMin = roundDownMinute(now);
    if (logger.isDebugEnabled()) {
        logger.debug("Cleaning up sessions expiring at " + new Date(prevMin));
    }
    // 根据整分时间获取过期键集合，如：spring:session:expirations:1439245080000
    String expirationKey = getExpirationKey(prevMin);
    // 获取所有的所有的过期session
    Set<Object> sessionsToExpire = this.redis.boundSetOps(expirationKey).members();
    // 删除过期Session键集合
    this.redis.delete(expirationKey);
    // touch访问所有已经过期的session，触发Redis键空间通知消息
    for (Object session : sessionsToExpire) {
        String sessionKey = getSessionKey((String) session);
        touch(sessionKey);
    }
}
```
将时间戳滚动至整分
```
static long roundDownMinute(long timeInMs) {
    Calendar date = Calendar.getInstance();
    date.setTimeInMillis(timeInMs);
    // 清理时间错的秒位和毫秒位
    date.clear(Calendar.SECOND);
    date.clear(Calendar.MILLISECOND);
    return date.getTimeInMillis();
}
```
获取过期Session的集合
```
String getExpirationKey(long expires) {
    return this.redisSession.getExpirationsKey(expires);
}

// 如：spring:session:expirations:1439245080000
String getExpirationsKey(long expiration) {
    return this.keyPrefix + "expirations:" + expiration;
}
```
调用Redis的Exists命令，访问过期Session键，触发Redis键空间消息
```
/**
 * By trying to access the session we only trigger a deletion if it the TTL is
 * expired. This is done to handle
 * https://github.com/spring-projects/spring-session/issues/93
 *
 * @param key the key
 */
private void touch(String key) {
    this.redis.hasKey(key);
}
```
总结
至此Spring-Session的Session事件通知模块就已经很清晰：

* Redis键空间Session事件源：Session创建通道/Session删除通道/Session过期通道
* Spring-Session中的RedisOperationsSessionRepository消息监听器监听Redis的事件类型
* RedisOperationsSessionRepository负责将其传播至ApplicationEventPublisher
* ApplicationEventPublisher将其包装成ApplicationEvent类型的Session Event发布
* ApplicationListener监听Session Event，处理相应逻辑


