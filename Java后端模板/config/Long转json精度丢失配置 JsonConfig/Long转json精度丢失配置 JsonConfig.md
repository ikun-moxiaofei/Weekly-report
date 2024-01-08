
```java
// 使用@JsonComponent注解标记该类是一个JSON组件，Spring Boot会自动扫描并注册该类。  
@JsonComponent  
public class JsonConfig {  
  
    /**  
     * 添加 Long 转 json 精度丢失的配置  
     */  
    // 使用@Bean注解标记该方法将返回一个对象，该对象会被Spring的IoC容器管理，并注册到Spring的应用上下文中。  
    @Bean  
    public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {  
        // 使用Jackson2ObjectMapperBuilder构建一个ObjectMapper对象，该对象用于JSON的序列化和反序列化。  
        // createXmlMapper(false)表示不创建用于XML的ObjectMapper。  
        ObjectMapper objectMapper = builder.createXmlMapper(false).build();  
          
        // 创建一个SimpleModule对象，用于添加自定义的序列化和反序列化方法。  
        SimpleModule module = new SimpleModule();  
          
        // 为Long.class添加一个序列化方法，使用ToStringSerializer将Long对象转换为字符串。  
        module.addSerializer(Long.class, ToStringSerializer.instance);  
          
        // 为Long.TYPE添加一个序列化方法，使用ToStringSerializer将Long基本类型转换为字符串。  
        module.addSerializer(Long.TYPE, ToStringSerializer.instance);  
          
        // 将上面创建的SimpleModule注册到ObjectMapper中，使得ObjectMapper使用该模块中的序列化和反序列化方法。  
        objectMapper.registerModule(module);  
          
        // 返回配置完成的ObjectMapper对象。  
        return objectMapper;  
    }  
}
```
使用构造函数注入或字段注入来获取ObjectMapper的实例。

```java
@Autowired  
private ObjectMapper objectMapper;
```
然后使用objectMapper.writeValueAsString()操作

```java
Long longValue = 123456789012345L; // 超出整数范围的长整型数  
String jsonString = objectMapper.writeValueAsString(longValue);
```
