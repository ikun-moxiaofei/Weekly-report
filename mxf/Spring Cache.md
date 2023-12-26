Spring Cache 是一个框架，实现了基于注解的缓存功能，只需要加一个注解，就能实现缓存功能。<br />Spring Cache 提供了一层抽象，底层可以切换不同的缓存实现，例如：

- EHCache
- Caffeine
- Redis(常用)

**起步依赖：**
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-cache</artifactId>  		            		       	 <version>2.7.3</version> 
</dependency>
```
<a name="8fe08533"></a>
#### 常用注解
在SpringCache中提供了很多缓存操作的注解，常见的是以下的几个：

| **注解** | **说明** |
| --- | --- |
| @EnableCaching  | 开启缓存注解功能，通常加在启动类上 |
| @Cacheable  | 在方法执行前先查询缓存中是否有数据，如果有数据，则直接返回缓存数据；如果没有缓存数据，调用方法并将方法返回值放到缓存中 |
| @CachePut  | 将方法的返回值放到缓存中 |
| @CacheEvict  | 将一条或多条数据从缓存中删除 |


在spring boot项目中，使用缓存技术只需在项目中导入相关缓存技术的依赖包，并在启动类上使用@EnableCaching开启缓存支持即可。

例如，使用Redis作为缓存技术，只需要导入Spring data Redis的maven坐标即可。

<a name="hhn0X"></a>
### 使用方法

- 先在引导类上添加@EnableCaching
- 在插入数据库(save)方法上加注解@CachePut  <br />当前UserController的save方法是用来保存用户信息的，在该用户信息保存到数据库的同时，也往缓存中缓存一份数据，可以在save方法上加上注解 @CachePut，用法如下：
```java
	/**
	* CachePut：将方法返回值放入缓存
	* value：缓存的名称，每个缓存名称下面可以有多个key
	* key：缓存的key
	*/
	@PostMapping
    @CachePut(value = "userCache", key = "#user.id")//key的生成：userCache::1
    public User save(@RequestBody User user){
        userMapper.insert(user);
        return user;
    }
```
key的其他写法：

   - user.id : #user指的是方法形参的名称, id指的是user的id属性 , 也就是使用user的id属性作为key ;
   - result.id : #result代表方法返回值，该表达式 代表以返回对象的id属性作为key ；
   - p0.id：#p0指的是方法中的第一个参数，id指的是第一个参数的id属性,也就是使用第一个参数的id属性作为key ;
   - a0.id：#a0指的是方法中的第一个参数，id指的是第一个参数的id属性,也就是使用第一个参数的id属性作为key ;
   - root.args[0].id:#root.args[0]指的是方法中的第一个参数，id指的是第一个参数的id属性,也就是使用第一个参数的id属性作为key ;
- 在查找(get)方法上加注解@Cacheable<br />在方法执行前，spring先查看缓存中是否有数据，如果有数据，则直接返回缓存数据；若没有数据，调用方法并将方法返回值放到缓存中
```java
	/**
	* Cacheable：在方法执行前spring先查看缓存中是否有数据，如果有数据，则直接返回缓存数据；若没有数据，	  *调用方法并将方法返回值放到缓存中
	* value：缓存的名称，每个缓存名称下面可以有多个key
	* key：缓存的key
	*/
	@GetMapping
    @Cacheable(cacheNames = "userCache",key="#id")
    public User getById(Long id){
        User user = userMapper.getById(id);
        return user; 
    }
```
key的其他写法与上头那个类似

- 在修改或者删除(delete)方法上加注解@CacheEvict<br />作用: 清理指定缓存

	value: 缓存的名称，每个缓存名称下面可以有多个key<br />	key: 缓存的key ----------> 支持Spring的表达式语言SPEL语法
```java
@DeleteMapping
@CacheEvict(cacheNames = "userCache",key = "#id")//删除某个key对应的缓存数据
public void deleteById(Long id){
    userMapper.deleteById(id);
}

@DeleteMapping("/delAll")
@CacheEvict(cacheNames = "userCache",allEntries = true)//删除userCache下所有的缓存数据
public void deleteAll(){
    userMapper.deleteAll();
}
```



