字符串<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/38886723/1703317547741-46e832f1-50c3-484c-99bb-82d4b155b594.png#averageHue=%23f8f7f7&clientId=u15956b9d-3147-4&from=paste&height=219&id=u82701778&originHeight=427&originWidth=1186&originalType=binary&ratio=1.9499999284744263&rotation=0&showTitle=false&size=101804&status=done&style=none&taskId=u946c849b-1676-4784-99d0-f041394894d&title=&width=608.2051505139601)<br />哈希<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/38886723/1703317486145-d3bef037-bc0d-48ec-998f-f95f71e26b34.png#averageHue=%23f7f7f6&clientId=u15956b9d-3147-4&from=paste&height=342&id=u6bf8554b&originHeight=666&originWidth=1225&originalType=binary&ratio=1.9499999284744263&rotation=0&showTitle=false&size=168643&status=done&style=none&taskId=ua73fbf14-8d46-4951-b93e-005f332434e&title=&width=628.2051512475558)

列表<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/38886723/1703317532155-441ed4c5-6639-413f-9e19-adeace7d5306.png#averageHue=%23f6f6f6&clientId=u15956b9d-3147-4&from=paste&height=335&id=u1013234a&originHeight=654&originWidth=900&originalType=binary&ratio=1.9499999284744263&rotation=0&showTitle=false&size=126012&status=done&style=none&taskId=u77a64e29-875b-46b1-a7bc-1d0b3448de1&title=&width=461.53847846759203)

集合<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/38886723/1703317568208-7e1f0bad-cea8-4f93-af29-443788bb9a54.png#averageHue=%23f6f6f6&clientId=u15956b9d-3147-4&from=paste&height=363&id=u4d134d7f&originHeight=708&originWidth=1135&originalType=binary&ratio=1.9499999284744263&rotation=0&showTitle=false&size=179532&status=done&style=none&taskId=u841995ef-c8fe-4301-bd61-36d1fa40e8a&title=&width=582.0513034007967)

有序集合<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/38886723/1703317608760-983d12ec-4286-4a4a-bc76-131c0f3c1649.png#averageHue=%23f4f4f4&clientId=u15956b9d-3147-4&from=paste&height=234&id=uc6bf4b9c&originHeight=456&originWidth=1393&originalType=binary&ratio=1.9499999284744263&rotation=0&showTitle=false&size=165839&status=done&style=none&taskId=u686c32ab-105b-4248-bc16-e0cca6cc9a9&title=&width=714.3590005615064)

通用命令<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/38886723/1703317632020-30059f8b-bdff-42b1-95da-9aa2eccdbb55.png#averageHue=%23f6f5f5&clientId=u15956b9d-3147-4&from=paste&height=231&id=skUz7&originHeight=451&originWidth=874&originalType=binary&ratio=1.9499999284744263&rotation=0&showTitle=false&size=100051&status=done&style=none&taskId=u16daebb9-572d-4085-96fc-15a1c6fcb27&title=&width=448.20514464519493)

<a name="f6Zoc"></a>
## java中使用Redis

- 导入maven坐标
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

- 配置数据源
```java
redis:
	host: localhost
    port: 6379    //主机和端口是默认值可以不用设置
    password:    //如果有设置密码需要配置
    database:  //redis 有许多数据库 可以选择默认为0
```

- 编写配置类，创建RedisTemplate对象
```java
@Configuration
@Slf4j
public class RedisConfiguration {
 
    @Bean
                                //先注入工厂对象
    public RedisTemplate redisTemplate(RedisConnectionFactory connectionFactory){
        RedisTemplate redisTemplate=new RedisTemplate();
            //连接工厂对象
        redisTemplate.setConnectionFactory(connectionFactory);
            //设置redis key的序列化器
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        return redisTemplate;
    }
}
```

- 通过RedisTemplateR对象操作Redis
   - **操作字符串类型数据**
```java
@Test
public void testString(){
    // set get setex setnx
    redisTemplate.opsForValue().set("name","小明");
    String city = (String) redisTemplate.opsForValue().get("name");
    System.out.println(city);
    redisTemplate.opsForValue().set("code","1234",3, TimeUnit.MINUTES);
    redisTemplate.opsForValue().setIfAbsent("lock","1");
    redisTemplate.opsForValue().setIfAbsent("lock","2");
}
```

   - ** 操作哈希类型数据**
```java
@Test
public void testHash(){
    //hset hget hdel hkeys hvals
    HashOperations hashOperations = redisTemplate.opsForHash();

    hashOperations.put("100","name","tom");
    hashOperations.put("100","age","20");

    String name = (String) hashOperations.get("100", "name");
    System.out.println(name);

    Set keys = hashOperations.keys("100");
    System.out.println(keys);

    List values = hashOperations.values("100");
    System.out.println(values);

    hashOperations.delete("100","age");
}
```

   - ** 操作列表类型数据**
```java
@Test
public void testList(){
    //lpush lrange rpop llen
    ListOperations listOperations = redisTemplate.opsForList();

    listOperations.leftPushAll("mylist","a","b","c");
    listOperations.leftPush("mylist","d");

    List mylist = listOperations.range("mylist", 0, -1);
    System.out.println(mylist);

    listOperations.rightPop("mylist");

    Long size = listOperations.size("mylist");
    System.out.println(size);
}
```

   - **操作集合类型数据**
```java
@Test
public void testSet(){
    //sadd smembers scard sinter sunion srem
    SetOperations setOperations = redisTemplate.opsForSet();

    setOperations.add("set1","a","b","c","d");
    setOperations.add("set2","a","b","x","y");

    Set members = setOperations.members("set1");
    System.out.println(members);

    Long size = setOperations.size("set1");
    System.out.println(size);

    Set intersect = setOperations.intersect("set1", "set2");
    System.out.println(intersect);

    Set union = setOperations.union("set1", "set2");
    System.out.println(union);

    setOperations.remove("set1","a","b");
}
```

   - **操作有序集合类型数据**
```java
@Test
public void testZset(){
    //zadd zrange zincrby zrem
    ZSetOperations zSetOperations = redisTemplate.opsForZSet();

    zSetOperations.add("zset1","a",10);
    zSetOperations.add("zset1","b",12);
    zSetOperations.add("zset1","c",9);

    Set zset1 = zSetOperations.range("zset1", 0, -1);
    System.out.println(zset1);

    zSetOperations.incrementScore("zset1","c",10);

    zSetOperations.remove("zset1","a","b");
}
```

   - **通用命令操作**
```java
@Test
public void testCommon(){
    //keys exists type del
    Set keys = redisTemplate.keys("*");
    System.out.println(keys);

    Boolean name = redisTemplate.hasKey("name");
    Boolean set1 = redisTemplate.hasKey("set1");

    for (Object key : keys) {
        DataType type = redisTemplate.type(key);
        System.out.println(type.name());
    }

    redisTemplate.delete("mylist");
}
```
