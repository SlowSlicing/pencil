[toc]

> Redis Cluster 搭建：[Redis Cluster 从零安装并详解](https://blog.csdn.net/wo18237095579/article/details/80895413)
> 　　此篇笔记以上放这篇笔记为基础

# pom 依赖

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

# yml 配置

```
spring:
  redis:
    cluster:
      # 各 Redis 节点信息
      nodes: 192.168.117.135:6379,192.168.117.135:6380,192.168.117.136:7379,192.168.117.136:7380,192.168.117.137:8379,192.168.117.137:8380
      # 执行命令超时时间
      command-timeout: 15000
      # 重试次数
      max-attempts: 5
      # 跨集群执行命令时要遵循的最大重定向数量
      max-redirects: 3
      # 连接池最大连接数（使用负值表示没有限制）
      max-active: 16
      # 连接池最大阻塞等待时间（使用负值表示没有限制）
      max-wait: -1
      # 连接池中的最大空闲连接
      max-idle: 8
      # 连接池中的最小空闲连接
      min-idle: 0
      # 是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
      test-on-borrow: true
```

# 配置类

### RedisProperties.java

```
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * @Author：大漠知秋
 * @Description：Redis 基本环境信息
 * @CreateDate：13:48 2018/7/4
 */
@Component
@ConfigurationProperties(prefix = "spring.redis.cluster")
@Data
public class RedisProperties {

    private String nodes;

    private Integer commandTimeout;

    private Integer maxAttempts;

    private Integer maxRedirects;

    private Integer maxActive;

    private Integer maxWait;

    private Integer maxIdle;

    private Integer minIdle;

    private boolean testOnBorrow;

}
```

### RedisConfig.java

```
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisNode;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import redis.clients.jedis.JedisCluster;
import redis.clients.jedis.JedisPoolConfig;

import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.List;

/**
 * @Author：大漠知秋
 * @Description：Redis 配置类
 * @CreateDate：13:48 2018/7/4
 */
@Configuration
@ConditionalOnClass(JedisCluster.class)
public class RedisConfig {

    @Resource
    private RedisProperties redisProperties;

    /**
     * 配置 Redis 连接池信息
     */
    @Bean
    public JedisPoolConfig getJedisPoolConfig() {
        JedisPoolConfig jedisPoolConfig =new JedisPoolConfig();
        jedisPoolConfig.setMaxIdle(redisProperties.getMaxIdle());
        jedisPoolConfig.setMaxWaitMillis(redisProperties.getMaxWait());
        jedisPoolConfig.setTestOnBorrow(redisProperties.isTestOnBorrow());

        return jedisPoolConfig;
    }

    /**
     * 配置 Redis Cluster 信息
     */
    @Bean
    public RedisClusterConfiguration getJedisCluster() {
        RedisClusterConfiguration redisClusterConfiguration = new RedisClusterConfiguration();
        redisClusterConfiguration.setMaxRedirects(redisProperties.getMaxRedirects());

        List<RedisNode> nodeList = new ArrayList<>();

        String[] cNodes = redisProperties.getNodes().split(",");
        //分割出集群节点
        for(String node : cNodes) {
            String[] hp = node.split(":");
            nodeList.add(new RedisNode(hp[0], Integer.parseInt(hp[1])));
        }
        redisClusterConfiguration.setClusterNodes(nodeList);

        return redisClusterConfiguration;
    }

    /**
     * 配置 Redis 连接工厂
     */
    @Bean
    public JedisConnectionFactory getJedisConnectionFactory(RedisClusterConfiguration redisClusterConfiguration, JedisPoolConfig jedisPoolConfig) {
        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory(redisClusterConfiguration, jedisPoolConfig);
        return jedisConnectionFactory;
    }

    /**
     * 设置数据存入redis 的序列化方式
     *  redisTemplate序列化默认使用的jdkSerializeable
     *  存储二进制字节码，导致key会出现乱码，所以自定义序列化类
     */
    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.afterPropertiesSet();

        return redisTemplate;
    }

}
```

# 对 RedisTemplate 的封装

### 接口 BaseRedisDaoInter<K, V>

```
import org.springframework.data.redis.connection.DataType;
import org.springframework.data.redis.core.ZSetOperations;

import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.TreeSet;

/**
 * @Author：大漠知秋
 * @Description：操作Redis Dao接口
 * @CreateDate：14:56 2018/3/8
 */
public interface BaseRedisDaoInter<K, V> {

    /**
     * 获取Redis中所有的键的key
     *
     * @return
     */
    Set<K> getAllKeys();

    /**
     * 获取所有的普通key-value
     * @return
     */
    Map<K, V> getAllString();

    /**
     * 获取所有的Set -key-value
     * @return
     */
    Map<K, Set<V>> getAllSet();

    /**
     * 获取所有的List -key-value
     * @return
     */
    Map<K, List<V>> getAllList();

    /**
     * 获取所有的Map -key-value
     * @return
     */
    Map<K, Map<K, V>> getAllMap();

    /**
     * 添加一个list
     *
     * @param key 键
     * @param objectList 值集合
     */
    void addList(K key, List<V> objectList);

    /**
     * 向list中增加值
     *
     * @param key 键
     * @param obj 值
     * @return 返回在list中的下标
     */
    long addList(K key, V obj);

    /**
     * 向list中增加 多个值
     *
     * @param key 键
     * @param obj 多个对象可变数组
     * @return 返回在list中的下标
     */
    long addList(K key, V... obj);

    /**
     * 获取list
     *
     * @param key List的key
     * @param s 开始下标
     * @param e 结束的下标
     * @return 数据集合
     */
    List<V> getList(K key, long s, long e);

    /**
     * 获取完整的list
     *
     * @param key 键
     * @return 数据集合
     */
    List<V> getList(K key);

    /**
     * 获取list集合中元素的个数
     *
     * @param key 键
     * @return 集合的个数
     */
    long getListSize(K key);

    /**
     * 移除list中某值集合
     *  移除list中 count个value为object的值,并且返回移除的数量,
     *  如果count为0,或者大于list中为value为object数量的总和,
     *  那么移除所有value为object的值,并且返回移除数量
     *
     * @param key 键
     * @param object 值
     * @return 返回移除数量
     */
    long removeListValue(K key, V object);

    /**
     * 移除list中某值
     *
     * @param key 键
     * @param object 值
     * @return 返回移除数量
     */
    long removeListValue(K key, V... object);

    /**
     * 批量删除key对应的value
     *
     * @param keys 键
     */
    void remove(final K... keys);

    /**
     * 删除数据
     *  根据key精确匹配删除
     *
     * @param key
     */
    void remove(final K key);

    /**
     * 设置Set的过期时间
     *
     * @param key 键
     * @param time 过期时间长度
     * @return 是否成功
     */
    Boolean setSetExpireTime(String key, Long time);

    /**
     * 获取所有的ZSet正序  -key-value 不获取权重值
     *
     * @return Map集合
     */
    Map<K, Set<V>> getAllZSetReverseRange();

    /**
     * 获取所有的ZSet倒序  -key-value 不获取权重值
     *
     * @return Map集合
     */
    Map<K, Set<V>> getAllZSetRange();

    /**
     * 通过分数删除ZSet中的值
     *
     * @param key 键
     * @param s 最小
     * @param e 最大
     */
    void removeZSetRangeByScore(String key, double s, double e);

    /**
     * 设置ZSet的过期时间
     *
     * @param key 键
     * @param time 过期时间 -单位s
     * @return 是否成功
     */
    Boolean setZSetExpireTime(String key, Long time);

    /**
     * 判断缓存中是否有key对应的value
     *
     * @param key 键
     * @return 是否存在
     */
    boolean exists(final K key);

    /**
     * 读取String缓存可以是对象
     *
     * @param key 键
     * @return 值
     */
    V get(final K key);

    /**
     * 读取String缓存集合 可以是对象
     *
     * @param key 键可变数组
     * @return 值集合
     */
    List<V> get(final K... key);

    /**
     * 读取缓存 可以是对象 根据正则表达式匹配
     *
     * @param regKey 正则键
     * @return 值集合
     */
    List<Object> getByRegular(final K regKey);

    /**
     * 写入缓存 可以是对象
     *
     * @param key 键
     * @param value 值
     */
    void set(final K key, V value);

    /**
     * 写入缓存
     *
     * @param key 键
     * @param value 值
     * @param expireTime 过期时间 -单位s
     */
    void set(final K key, V value, Long expireTime);

    /**
     * 设置一个key的过期时间（单位：秒）
     *
     * @param key 键
     * @param expireTime 过期时间 -单位s
     * @return 是否成功
     */
    boolean setExpireTime(K key, Long expireTime);

    /**
     * 获取key的类型
     *
     * @param key 键
     * @return 键的类型
     */
    DataType getType(K key);

    /**
     * 删除Map中的某个对象
     *
     * @param key Map对应的key
     * @param field Map中该对象的key
     */
    void removeMapField(K key, V... field);

    /**
     * 获取Map对象
     *
     * @param key Map对应的key
     * @return Map数据
     */
    Map<K, V> getMap(K key);

    /**
     * 获取Map对象的长度
     *
     * @param key Map对应的key
     * @return Map的长度
     */
    Long getMapSize(K key);

    /**
     * 获取Map缓存中的某个对象
     *
     * @param key Map对应的key
     * @param field Map中该对象的key
     * @return Map中的Value
     */
    <T> T getMapField(K key, K field);

    /**
     * 判断Map中对应key的key是否存在
     *
     * @param key Map对应的key
     * @param field Map中的key
     * @return 是否存在
     */
    Boolean hasMapKey(K key, K field);

    /**
     * 获取Map对应key的value
     *
     * @param key Map对应的key
     * @return Map中值的集合
     */
    List<V> getMapFieldValue(K key);

    /**
     * 获取Map的key
     *
     * @param key Map对应的key
     * @return Map中key的集合
     */
    Set<V> getMapFieldKey(K key);

    /**
     * 添加Map
     *
     * @param key 键
     * @param map 值
     */
    void addMap(K key, Map<K, V> map);

    /**
     * 向key对应的Map中添加缓存对象
     *
     * @param key cache对象key
     * @param field Map对应的key
     * @param value 值
     */
    void addMap(K key, K field, Object value);

    /**
     * 向key对应的Map中添加缓存对象，带过期时间
     *
     * @param key cache对象key
     * @param field Map对应的key
     * @param time 过期时间-整个Map的过期时间
     * @param value 值
     */
    void addMap(K key, K field, V value, long time);

    /**
     * 向Set中加入对象
     *
     * @param key 对象key
     * @param obj 值
     */
    void addSet(K key, V... obj);

    /**
     * 移除Set中的某些值
     *
     * @param key 对象key
     * @param obj 值
     */
    long removeSetValue(K key, V obj);

    /**
     * 移除Set中的某些值可变数组
     *
     * @param key 对象key
     * @param obj 值
     */
    long removeSetValue(K key, V... obj);

    /**
     * 获取Set的对象数
     *
     * @param key 对象key
     */
    long getSetSize(K key);

    /**
     * 判断Set中是否存在这个值
     *
     * @param key 对象key
     */
    Boolean hasSetValue(K key, V obj);

    /**
     * 获得整个Set
     *
     * @param key 对象key
     */
    Set<V> getSet(K key);

    /**
     * 获得Set并集
     *
     * @param key 键
     * @param otherKey 其他的键
     * @return 获取两个Set类型值的集合
     */
    Set<V> getSetUnion(K key, K otherKey);

    /**
     * 获得Set并集
     *
     * @param key 键
     * @param set 一个Set数据
     * @return 缓存中的Set数据和传入的Set数据的并集
     */
    Set<V> getSetUnion(K key, Set<Object> set);

    /**
     * 获得Set交集
     *
     * @param key 键
     * @param otherKey 其他的键
     * @return 获取两个Set类型值的交接
     */
    Set<V> getSetIntersect(K key, K otherKey);

    /**
     * 获得Set交集
     *
     * @param key 键
     * @param set 一个Set数据
     * @return 缓存中的Set数据和传入的Set数据的交集
     */
    Set<V> getSetIntersect(K key, Set<Object> set);

    /**
     * 模糊移除 支持*号等匹配移除
     *
     * @param blear 模糊参数
     */
    void removeBlear(K blear);

    /**
     * 多个模糊条件移除 支持*号等匹配移除
     *
     * @param blears 模糊参数可变数组
     */
    void removeBlear(K... blears);

    /**
     * 修改key名 如果不存在该key或者没有修改成功返回false
     *
     * @param oldKey 原有的key名称
     * @param newKey 新key名称
     * @return 是否修改成功
     */
    Boolean renameIfAbsent(String oldKey, String newKey);

    /**
     * 根据正则表达式来移除key-value
     *
     * @param blears 正则
     */
    void removeByRegular(String blears);

    /**
     * 根据正则表达式来移除key-value
     *
     * @param blears 正则可变数组
     */
    void removeByRegular(String... blears);

    /**
     * 根据正则表达式来移除 Map中的key-value
     *
     * @param key 键
     * @param blear 正则
     */
    void removeMapFieldByRegular(K key, K blear);

    /**
     * 根据正则表达式来移除 Map中的key-value
     *
     * @param key 键
     * @param blears 正则可变数组
     */
    void removeMapFieldByRegular(K key, K... blears);

    /**
     * 移除key 对应的value
     *
     * @param key 键
     * @param value 值可变数组
     * @return 移除了多少个
     */
    Long removeZSetValue(K key, V... value);

    /**
     * 移除key ZSet
     *
     * @param key 键
     */
    void removeZSet(K key);

    /**
     *删除，键为K的集合，索引start<=index<=end的元素子集
     *
     * @param key 键
     * @param start 开始删除索引
     * @param end 结束删除索引
     */
    void removeZSetRange(K key, Long start, Long end);

    /**
     * 并集 将key对应的集合和key1对应的集合合并到key2中
     *  如果分数相同的值，都会保留
     *  原来key2的值会被覆盖
     *
     * @param key 键
     * @param key1 键1
     * @param key2 键2
     */
    void setZSetUnionAndStore(String key, String key1, String key2);

    /**
     * 获取整个有序集合ZSET，正序
     *
     * @param key 键
     * @return 有序集合
     */
    <T> T getZSetRange(K key);

    /**
     * 获取有序集合ZSET
     *  键为K的集合，索引start<=index<=end的元素子集，正序
     *
     * @param key 键
     * @param start 开始位置
     * @param end 结束位置
     * @return 有序集合
     */
    <T> T getZSetRange(K key, long start, long end);

    /**
     * 获取整个有序集合ZSET，倒序
     *
     * @param key 键
     * @return 有序集合
     */
    Set<Object> getZSetReverseRange(K key);

    /**
     * 获取有序集合ZSET
     *  键为K的集合，索引start<=index<=end的元素子集，倒序
     *
     * @param key 键
     * @param start 开始位置
     * @param end 结束位置
     * @return 有序集合
     */
    Set<V> getZSetReverseRange(K key, long start, long end);

    /**
     * 通过分数(权值)获取ZSET集合 正序 -从小到大
     *
     * @param key 键
     * @param start 开始
     * @param end 结束
     * @return 正序集合
     */
    Set<V> getZSetRangeByScore(String key, double start, double end);

    /**
     * 通过分数(权值)获取ZSET集合 倒序 -从大到小
     *
     * @param key 键
     * @param start 开始
     * @param end 结束
     * @return 倒序集合
     */
    Set<V> getZSetReverseRangeByScore(String key, double start, double end);

    /**
     * 键为K的集合，索引start<=index<=end的元素子集
     *  返回泛型接口（包括score和value），正序
     *
     * @param key 键
     * @param start 开始
     * @param end 结束
     * @return 正序集合
     */
    Set<ZSetOperations.TypedTuple<V>> getZSetRangeWithScores(K key, long start, long end);

    /**
     * 键为K的集合，索引start<=index<=end的元素子集
     *  返回泛型接口（包括score和value），倒序
     *
     * @param key 键
     * @param start 开始
     * @param end 结束
     * @return 倒序集合
     */
    Set<ZSetOperations.TypedTuple<V>> getZSetReverseRangeWithScores(K key, long start, long end);

    /**
     * 键为K的集合
     *  返回泛型接口（包括score和value），正序
     *
     * @param key 键
     * @return 正序集合
     */
    Set<ZSetOperations.TypedTuple<V>> getZSetRangeWithScores(K key);
    /**
     * 键为K的集合
     *  返回泛型接口（包括score和value），倒序
     *
     * @param key 键
     * @return 倒序集合
     */
    Set<ZSetOperations.TypedTuple<V>> getZSetReverseRangeWithScores(K key);

    /**
     * 键为K的集合，sMin<=score<=sMax的元素个数
     *
     * @param key 键
     * @param sMin 最小
     * @param sMax 最大
     * @return 个数
     */
    long getZSetCountSize(K key, double sMin, double sMax);

    /**
     * 获取Zset 键为K的集合元素个数
     *
     * @param key 键
     * @return 集合的长度
     */
    long getZSetSize(K key);

    /**
     * 获取键为K的集合，value为obj的元素分数
     *
     * @param key 键
     * @param value 值
     * @return 分数
     */
    double getZSetScore(K key, V value);

    /**
     * 元素分数增加，delta是增量
     *
     * @param key 键
     * @param value 值
     * @param delta 增分
     * @return 分数
     */
    double incrementZSetScore(K key, V value, double delta);

    /**
     * 添加有序集合ZSET
     *  默认按照score升序排列，存储格式K(1)==V(n)，V(1)=S(1)
     *
     * @param key 键
     * @param score 分数
     * @param value 值
     * @return 是否成功
     */
    Boolean addZSet(String key, double score, Object value);

    /**
     * 添加有序集合ZSET
     *
     * @param key 键
     * @param value 值
     * @return 长度
     */
    Long addZSet(K key, TreeSet<V> value);

    /**
     * 添加有序集合ZSET
     *
     * @param key 键
     * @param score 分数
     * @param value 值
     * @return 是否成功
     */
    Boolean addZSet(K key, double[] score, Object[] value);

}
```

### 实现 BaseRedisDao

```
import com.lynchj.rediscluster.dao.BaseRedisDaoInter;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.redis.connection.DataType;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ZSetOperations;
import org.springframework.stereotype.Repository;
import org.springframework.util.CollectionUtils;

import javax.annotation.Resource;
import java.util.*;
import java.util.concurrent.TimeUnit;
import java.util.regex.Pattern;

/**
 * @Author：大漠知秋
 * @Description：操作Redis Dao接口
 *                  增删改 -不能在这里面抓取异常 -因为可能有事务处理
 * @CreateDate：14:56 2018/3/8
 */
@Repository
@Slf4j
public class BaseRedisDao implements BaseRedisDaoInter<String, Object> {

    @Resource(name = "redisTemplate")
    private RedisTemplate redisTemplate;

    /**  出异常，重复操作的次数 */
    private static final Integer TIMES = 3;

    @Override
    public Set<String> getAllKeys() {
        return redisTemplate.keys("*");
    }

    @Override
    public Map<String, Object> getAllString() {
        Set<String> stringSet = getAllKeys();
        Map<String, Object> map = new HashMap<String, Object>();
        Iterator<String> iterator = stringSet.iterator();
        while (iterator.hasNext()) {
            String k = iterator.next();
            if (getType(k) == DataType.STRING) {
                map.put(k, get(k));
            }
        }
        return map;
    }

    @Override
    public Map<String, Set<Object>> getAllSet() {
        Set<String> stringSet = getAllKeys();
        Map<String, Set<Object>> map = new HashMap<String, Set<Object>>();
        Iterator<String> iterator = stringSet.iterator();
        while (iterator.hasNext()) {
            String k = iterator.next();
            if (getType(k) == DataType.SET) {
                map.put(k, getSet(k));
            }
        }
        return map;
    }

    @Override
    public Map<String, Set<Object>> getAllZSetRange() {
        Set<String> stringSet = getAllKeys();
        Map<String, Set<Object>> map = new HashMap<String, Set<Object>>();
        Iterator<String> iterator = stringSet.iterator();
        while (iterator.hasNext()) {
            String k = iterator.next();
            if (getType(k) == DataType.ZSET) {
                log.debug("k:" + k);
                map.put(k, getZSetRange(k));
            }
        }
        return map;
    }

    @Override
    public Map<String, Set<Object>> getAllZSetReverseRange() {
        Set<String> stringSet = getAllKeys();
        Map<String, Set<Object>> map = new HashMap<String, Set<Object>>();
        Iterator<String> iterator = stringSet.iterator();
        while (iterator.hasNext()) {
            String k = iterator.next();
            if (getType(k) == DataType.ZSET) {
                map.put(k, getZSetReverseRange(k));
            }
        }
        return map;
    }

    @Override
    public Map<String, List<Object>> getAllList() {
        Set<String> stringSet = getAllKeys();
        Map<String, List<Object>> map = new HashMap<String, List<Object>>();
        Iterator<String> iterator = stringSet.iterator();
        while (iterator.hasNext()) {
            String k = iterator.next();
            if (getType(k) == DataType.LIST) {
                map.put(k, getList(k));
            }
        }
        return map;
    }

    @Override
    public Map<String, Map<String, Object>> getAllMap() {
        Set<String> stringSet = getAllKeys();
        Map<String, Map<String, Object>> map = new HashMap<String, Map<String, Object>>();
        Iterator<String> iterator = stringSet.iterator();
        while (iterator.hasNext()) {
            String k = iterator.next();
            if (getType(k) == DataType.HASH) {
                map.put(k, getMap(k));
            }
        }
        return map;
    }

    @Override
    public void addList(String key, List<Object> objectList) {
        for (Object obj : objectList) {
            addList(key, obj);
        }
    }

    @Override
    public long addList(String key, Object obj) {
        return redisTemplate.boundListOps(key).rightPush(obj);
    }

    @Override
    public long addList(String key, Object... obj) {
        return redisTemplate.boundListOps(key).rightPushAll(obj);
    }

    @Override
    public List<Object> getList(String key, long s, long e) {
        return redisTemplate.boundListOps(key).range(s, e);
    }

    @Override
    public List<Object> getList(String key) {
        return redisTemplate.boundListOps(key).range(0, getListSize(key));
    }

    @Override
    public long getListSize(String key) {
        return redisTemplate.boundListOps(key).size();
    }

    @Override
    public long removeListValue(String key, Object object) {
        return redisTemplate.boundListOps(key).remove(0, object);
    }

    @Override
    public long removeListValue(String key, Object... objects) {
        long r = 0;
        for (Object object : objects) {
            r += removeListValue(key, object);
        }
        return r;
    }

    @Override
    public void remove(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                remove(key[0]);
            } else {
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }

    @Override
    public void removeBlear(String... blears) {
        for (String blear : blears) {
            removeBlear(blear);
        }
    }

    @Override
    public Boolean renameIfAbsent(String oldKey, String newKey) {
        return redisTemplate.renameIfAbsent(oldKey, newKey);
    }

    @Override
    public void removeBlear(String blear) {
        redisTemplate.delete(redisTemplate.keys(blear));
    }

    @Override
    public void removeByRegular(String... blears) {
        for (String blear : blears) {
            removeBlear(blear);
        }
    }

    @Override
    public void removeByRegular(String blear) {
        Set<String> stringSet = getAllKeys();
        for (String s : stringSet) {
            if (Pattern.compile(blear).matcher(s).matches()) {
                redisTemplate.delete(s);
            }
        }
    }

    @Override
    public void removeMapFieldByRegular(String key, String... blears) {
        for (String blear : blears) {
            removeMapFieldByRegular(key, blear);
        }
    }

    @Override
    public void removeMapFieldByRegular(String key, String blear) {
        Map<String, Object> map = getMap(key);
        Set<String> stringSet = map.keySet();
        for (String s : stringSet) {
            if (Pattern.compile(blear).matcher(s).matches()) {
                redisTemplate.boundHashOps(key).delete(s);
            }
        }
    }

    @Override
    public Long removeZSetValue(String key, Object... value) {
        return redisTemplate.boundZSetOps(key).remove(value);
    }

    @Override
    public void removeZSet(String key) {
        removeZSetRange(key, 0L, getZSetSize(key));
    }

    @Override
    public void removeZSetRange(String key, Long start, Long end) {
        redisTemplate.boundZSetOps(key).removeRange(start, end);
    }

    @Override
    public void setZSetUnionAndStore(String key, String key1, String key2) {
        redisTemplate.boundZSetOps(key).unionAndStore(key1, key2);
    }

    @Override
    public Set<Object> getZSetRange(String key) {
        return getZSetRange(key, 0, getZSetSize(key));
    }

    @Override
    public Set<Object> getZSetRange(String key, long s, long e) {
        return redisTemplate.boundZSetOps(key).range(s, e);
    }

    @Override
    public Set<Object> getZSetReverseRange(String key) {
        return getZSetReverseRange(key, 0, getZSetSize(key));
    }

    @Override
    public Set<Object> getZSetReverseRange(String key, long start, long end) {
        return redisTemplate.boundZSetOps(key).reverseRange(start, end);
    }

    @Override
    public Set<Object> getZSetRangeByScore(String key, double start, double end) {
        return redisTemplate.boundZSetOps(key).rangeByScore(start, end);
    }

    @Override
    public Set<Object> getZSetReverseRangeByScore(String key, double start, double end) {
        return redisTemplate.boundZSetOps(key).reverseRangeByScore(start, end);
    }

    @Override
    public Set<ZSetOperations.TypedTuple<Object>> getZSetRangeWithScores(String key, long start, long end) {
        return redisTemplate.boundZSetOps(key).rangeWithScores(start, end);
    }

    @Override
    public Set<ZSetOperations.TypedTuple<Object>> getZSetReverseRangeWithScores(String key, long start, long end) {
        return redisTemplate.boundZSetOps(key).reverseRangeWithScores(start, end);
    }

    @Override
    public Set<ZSetOperations.TypedTuple<Object>> getZSetRangeWithScores(String key) {
        return getZSetRangeWithScores(key, 0, getZSetSize(key));
    }

    @Override
    public Set<ZSetOperations.TypedTuple<Object>> getZSetReverseRangeWithScores(String key) {
        return getZSetReverseRangeWithScores(key, 0, getZSetSize(key));
    }

    @Override
    public long getZSetCountSize(String key, double sMin, double sMax) {
        return redisTemplate.boundZSetOps(key).count(sMin, sMax);
    }

    @Override
    public long getZSetSize(String key) {
        return redisTemplate.boundZSetOps(key).size();
    }

    @Override
    public double getZSetScore(String key, Object value) {
        return redisTemplate.boundZSetOps(key).score(value);
    }

    @Override
    public double incrementZSetScore(String key, Object value, double delta) {
        return redisTemplate.boundZSetOps(key).incrementScore(value, delta);
    }

    @Override
    public Boolean addZSet(String key, double score, Object value) {
        return redisTemplate.boundZSetOps(key).add(value, score);
    }

    @Override
    public Long addZSet(String key, TreeSet<Object> value) {
        return redisTemplate.boundZSetOps(key).add(value);
    }

    @Override
    public Boolean addZSet(String key, double[] score, Object[] value) {
        if (score.length != value.length) {
            return false;
        }
        for (int i = 0; i < score.length; i++) {
            if (addZSet(key, score[i], value[i]) == false) {
                return false;
            }
        }
        return true;
    }

    @Override
    public void remove(String key) {
        if (exists(key)) {
            redisTemplate.delete(key);
        }
    }

    @Override
    public void removeZSetRangeByScore(String key, double s, double e) {
        redisTemplate.boundZSetOps(key).removeRangeByScore(s, e);
    }

    @Override
    public Boolean setSetExpireTime(String key, Long time) {
        return redisTemplate.boundSetOps(key).expire(time, TimeUnit.SECONDS);
    }

    @Override
    public Boolean setZSetExpireTime(String key, Long time) {
        return redisTemplate.boundZSetOps(key).expire(time, TimeUnit.SECONDS);
    }

    @Override
    public boolean exists(String key) {
        return redisTemplate.hasKey(key);
    }

    @Override
    public Object get(String key) {
        return redisTemplate.boundValueOps(key).get();
    }

    @Override
    public List<Object> get(String... keys) {
        List<Object> list = new ArrayList<Object>();
        for (String key : keys) {
            list.add(get(key));
        }
        return list;
    }

    @Override
    public List<Object> getByRegular(String regKey) {
        Set<String> stringSet = getAllKeys();
        List<Object> objectList = new ArrayList<Object>();
        for (String s : stringSet) {
            if (Pattern.compile(regKey).matcher(s).matches() && getType(s) == DataType.STRING) {
                objectList.add(get(s));
            }
        }
        return objectList;
    }

    @Override
    public void set(String key, Object value) {
        redisTemplate.boundValueOps(key).set(value);
    }

    @Override
    public void set(String key, Object value, Long expireTime) {
        redisTemplate.boundValueOps(key).set(value, expireTime, TimeUnit.SECONDS);
    }

    @Override
    public boolean setExpireTime(String key, Long expireTime) {
        return redisTemplate.expire(key, expireTime, TimeUnit.SECONDS);
    }


    @Override
    public DataType getType(String key) {
        return redisTemplate.type(key);
    }


    @Override
    public void removeMapField(String key, Object... field) {
        redisTemplate.boundHashOps(key).delete(field);
    }

    @Override
    public Long getMapSize(String key) {
        return redisTemplate.boundHashOps(key).size();
    }

    @Override
    public Map<String, Object> getMap(String key) {
        return redisTemplate.boundHashOps(key).entries();
    }

    @Override
    public <T> T getMapField(String key, String field) {
        return (T) redisTemplate.boundHashOps(key).get(field);
    }

    @Override
    public Boolean hasMapKey(String key, String field) {
        return redisTemplate.boundHashOps(key).hasKey(field);
    }

    @Override
    public List<Object> getMapFieldValue(String key) {
        return redisTemplate.boundHashOps(key).values();
    }

    @Override
    public Set<Object> getMapFieldKey(String key) {
        return redisTemplate.boundHashOps(key).keys();
    }

    @Override
    public void addMap(String key, Map<String, Object> map) {
        redisTemplate.boundHashOps(key).putAll(map);
    }

    @Override
    public void addMap(String key, String field, Object value) {
        redisTemplate.boundHashOps(key).put(field, value);
    }

    @Override
    public void addMap(String key, String field, Object value, long time) {
        redisTemplate.boundHashOps(key).put(field, value);
        redisTemplate.boundHashOps(key).expire(time, TimeUnit.SECONDS);
    }

    @Override
    public void addSet(String key, Object... obj) {
        redisTemplate.boundSetOps(key).add(obj);
    }

    @Override
    public long removeSetValue(String key, Object obj) {
        return redisTemplate.boundSetOps(key).remove(obj);
    }

    @Override
    public long removeSetValue(String key, Object... obj) {
        if (obj != null && obj.length > 0) {
            return redisTemplate.boundSetOps(key).remove(obj);
        }
        return 0L;
    }

    @Override
    public long getSetSize(String key) {
        return redisTemplate.boundSetOps(key).size();
    }

    @Override
    public Boolean hasSetValue(String key, Object obj) {
        Boolean boo = null;
        int t = 0;
        while (true) {
            try {
                boo = redisTemplate.boundSetOps(key).isMember(obj);
                break;
            } catch (Exception e) {
                log.error("key[" + key + "],obj[" + obj + "]判断Set中的值是否存在失败,异常信息:" + e.getMessage());
                t++;
            }
            if (t > TIMES) {
                break;
            }
        }
        log.info("key[" + key + "],obj[" + obj + "]是否存在,boo:" + boo);
        return boo;
    }

    @Override
    public Set<Object> getSet(String key) {
        return redisTemplate.boundSetOps(key).members();
    }

    @Override
    public Set<Object> getSetUnion(String key, String otherKey) {
        return redisTemplate.boundSetOps(key).union(otherKey);
    }

    @Override
    public Set<Object> getSetUnion(String key, Set<Object> set) {
        return redisTemplate.boundSetOps(key).union(set);
    }

    @Override
    public Set<Object> getSetIntersect(String key, String otherKey) {
        return redisTemplate.boundSetOps(key).intersect(otherKey);
    }

    @Override
    public Set<Object> getSetIntersect(String key, Set<Object> set) {
        return redisTemplate.boundSetOps(key).intersect(set);
    }

}
```

# 调用

　　在需要使用的类中注入 RedisTemplate 的包装 Dao 即可，如下：

```
@Resource
private BaseRedisDaoInter redisDaoInter;
```
