> 此篇笔记是以上篇笔记为基础：[Spring Boot 结合 Redis Cluster RedisTemplate](https://blog.csdn.net/wo18237095579/article/details/80925586)

# 关于 Redis Cache Manager 的微小修改

```
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.core.RedisTemplate;

import java.lang.reflect.Method;

@Configuration
@EnableCaching
public class RedisCacheConfig {

    /**
     * 重新配置 RedisCacheManager
     */
    @Bean
    public CacheManager cacheManager(RedisTemplate<Object, Object> redisTemplate) {
        RedisCacheManager rcm = new RedisCacheManager(redisTemplate);
        // 设置缓存过期时间，单位：秒，默认 永不过期
        rcm.setDefaultExpiration(100L);
        return rcm;
    }

	/**
     * 自定义缓存时，keyGenerator 的生成规则
     */
    @Bean
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... objects) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : objects) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }

}
```

# @EnableCaching 注解

　　在 Spring Boot 启动类上方开启 Redis 缓存支持

```
@SpringBootApplication
// 开启 Redis 缓存支持
@EnableCaching 
public class RedisClusterApplication {

	public static void main(String[] args) {
		SpringApplication.run(RedisClusterApplication.class, args);
	}

}
```

# @Cacheable 注解

　　作用在方法上，指定可以缓存调用的方法。大白话就是：在哪个方法上面就针对哪个做缓存调用。

### cacheNames

* **指定缓存的名称**

### key

* **定义组成的 key 值，如果不定义，则使用全部的参数计算一个 key 值。可以使用 Spring El 表达式**

```
/**
 * cacheNames 设置缓存的值 
 * key：指定缓存的key，这是指参数id值。 key可以使用spEl表达式
 * 
 * @param id
 * @return
 */
@Cacheable(cacheNames = "book1", key = "#id")
public Book queryBookCacheable(String id){
    logger.info("queryBookCacheable,id={}",id);
    return repositoryBook.get(id);
}

/**
 * 这里使用另一个缓存存储缓存
 * 
 * @param id
 * @return
 */
@Cacheable(cacheNames = "book2", key = "#id")
public Book queryBookCacheable_2(String id){
    logger.info("queryBookCacheable_2,id={}",id);
    return repositoryBook.get(id);
}

/**
 * 缓存的key也可以指定对象的成员变量
 * 
 * @param qry
 * @return
 */
@Cacheable(cacheNames = "book1", key = "#qry.id")
public Book queryBookCacheableByBookQry(BookQry qry){
    logger.info("queryBookCacheableByBookQry,qry={}",qry);
    String id = qry.getId();
    Assert.notNull(id, "id can't be null!");
    String name = qry.getName();
    Book book = null;
    if(id != null){
        book = repositoryBook.get(id);
        if(book != null && !(name != null && book.getName().equals(name))){
            book = null;
        }
    }
    return book;
}
```

### keyGenerator

* **定义 key 生成的类，和 `key` 不能同时存在。笔记一开始已经的`微小修改`中已经有定义**

```
/**
 * 以上我们使用默认的keyGenerator，对应spring的SimpleKeyGenerator 
 *  如果你的使用很复杂，我们也可以自定义keyGenerator的生成key
 * 
 *  key和keyGenerator是互斥，如果同时制定会出异常
 *  The key and keyGenerator parameters are mutually exclusive and an operation specifying both will result in an exception.
 * 
 * @param id
 * @return
 */
@Cacheable(cacheNames="book3",  keyGenerator="keyGenerator")
public Book queryBookCacheableUseMyKeyGenerator(String id){
    logger.info("queryBookCacheableUseMyKeyGenerator,id={}",id);
    return repositoryBook.get(id);
}
```

### sync

* **如果设置sync = true：a. 如果缓存中没有数据，多个线程同时访问这个方法，则只有一个方法会执行到方法，其它方法需要等待; b. 如果缓存中已经有数据，则多个线程可以同时从缓存中获取数据**

```
/**
 * 如果设置 sync = true，
 *  如果缓存中没有数据，多个线程同时访问这个方法，则只有一个方法会执行到方法，其它方法需要等待
 *  如果缓存中已经有数据，则多个线程可以同时从缓存中获取数据
 * 
 * @param id
 * @return
 */
@Cacheable(cacheNames = "book3", sync = true)
public Book queryBookCacheableWithSync(String id) {
    logger.info("begin ... queryBookCacheableByBookQry,id={}",id);
    try {
        Thread.sleep(1000 * 2);
    } catch (InterruptedException e) {
    }
    logger.info("end ... queryBookCacheableByBookQry,id={}",id);
    return repositoryBook.get(id);
}
```

### condition 和 unless

* **condition：** 在执行方法前，condition 的值为 true，则缓存数据
* **unless：**在执行方法后，判断 unless ，如果值为 true，则不缓存数据
* conditon 和 unless 可以同时使用，则此时只缓存同时满足两者的记录

```
/**
 * 条件缓存：
 *  只有满足 condition 的请求才可以进行缓存，如果不满足条件，则跟方法没有 @Cacheable 注解的方法一样
 *  如下面只有id < 3 才进行缓存
 */
@Cacheable(cacheNames = "book11", condition = "T(java.lang.Integer).parseInt(#id) < 3")
public Book queryBookCacheableWithCondition(String id) {
    logger.info("queryBookCacheableByBookQry,id={}",id);
    return repositoryBook.get(id);
}

/**
 * 条件缓存：
 *  对不满足unless的记录，才进行缓存
 *  "unless expressions" are evaluated after the method has been called
 *  如下面：只对不满足返回 'T(java.lang.Integer).parseInt(#result.id) < 3' 的记录进行缓存
 */
@Cacheable(cacheNames = "book22", unless = "T(java.lang.Integer).parseInt(#result.id) < 3")
public Book queryBookCacheableWithUnless(String id) {
    logger.info("queryBookCacheableByBookQry,id={}",id);
    return repositoryBook.get(id);
}
```

# @CacheEvict 注解

　　删除缓存

### allEntries

* **allEntries = true：**清空缓存book1里的所有值
* **allEntries = false：**默认值，此时只删除key对应的值

```
/**
 * allEntries = true: 清空book1里的所有缓存
 */
@CacheEvict(cacheNames = "book1", allEntries = true)
public void clearBook1All(){
    logger.info("clearAll");
}

/**
 * 无 或者 allEntries = false: 对符合key条件的记录从缓存中book1移除
 */
@CacheEvict(cacheNames = "book1", key = "#id")
public void updateBook(String id, String name){
    logger.info("updateBook");
    Book book = repositoryBook.get(id);
    if(book != null){
        book.setName(name);
        book.setUpdate(new Date());
    }
}
```

# @CachePut 注解

　　每次执行都会执行方法，无论缓存里是否有值，同时使用新的返回值的替换缓存中的值。这里不同于`@Cacheable`；`@Cacheable`如果缓存没有值，则执行方法并缓存数据，如果缓存有值，则从缓存中获取值

```
@CachePut(cacheNames = "book1", key = "#id")
public Book queryBookCachePut(String id){
    logger.info("queryBookCachePut,id={}",id);
    return repositoryBook.get(id);
}
```

# @CacheConfig

　　**类级别的注解：**如果我们在此注解中定义 `cacheNames`，则此类中的所有方法上 `@Cacheable` 的`cacheNames` 默认都是此值。当然 `@Cacheable` 也可以重定义 `cacheNames` 的值

```
@Component
@CacheConfig(cacheNames="booksAll") 
public class BookService2 extends AbstractService {

    private static final Logger logger = LoggerFactory.getLogger(BookService2.class);

    /**
     * 此方法的@Cacheable没有定义cacheNames，则使用类上的注解@CacheConfig里的值 cacheNames
     * @param id
     * @return
     */
    @Cacheable(key="#id")
    public Book queryBookCacheable(String id){
        logger.info("queryBookCacheable,id={}",id);
        return repositoryBook.get(id);
    }

    /**
     * 此方法的@Cacheable有定义cacheNames，则使用此值覆盖类注解@CacheConfig里的值cacheNames
     * 
     * @param id
     * @return
     */
    @Cacheable(cacheNames="books_custom", key="#id")
    public Book queryBookCacheable2(String id){
        logger.info("queryBookCacheable2,id={}",id);
        return repositoryBook.get(id);
    }
}
```

> 参考：https://blog.csdn.net/hry2015/article/details/75451705
