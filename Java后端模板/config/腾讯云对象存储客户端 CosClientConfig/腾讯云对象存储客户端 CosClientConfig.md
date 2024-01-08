CosClientConfig
```java
// 声明该类是一个Spring配置类  
@Configuration  
// 使用@ConfigurationProperties注解，表示该类是Spring Boot的配置类，并指定前缀为"cos.client"，用于从配置文件中读取属性并自动填充到对应的字段中  
@ConfigurationProperties(prefix = "cos.client")  
// 使用@Data注解，它是Lombok库提供的一个注解，可以自动生成该类的getter、setter、equals、hashCode和toString方法，减少代码量  
@Data  
public class CosClientConfig {  
  
    /**  
     * 访问密钥  
     */  
    // 私有的字符串类型的accessKey字段，用于存储访问密钥  
    private String accessKey;  
  
    /**  
     * 密钥  
     */  
    // 私有的字符串类型的secretKey字段，用于存储密钥  
    private String secretKey;  
  
    /**  
     * 区域  
     */  
    // 私有的字符串类型的region字段，用于存储服务的区域信息  
    private String region;  
  
    /**  
     * 存储桶名称  
     */  
    // 私有的字符串类型的bucket字段，用于存储要操作的存储桶的名称  
    private String bucket;  
  
    // 使用@Bean注解，表示该方法将返回一个对象，该对象将由Spring的IoC容器管理，并注册到Spring的应用上下文中，通常用于配置类的结尾处，创建并返回一个Bean实例  
    @Bean  
    public COSClient cosClient() {  
        // 初始化用户的身份信息，传入accessKey和secretKey作为参数  
        // 创建一个COSCredentials实例，其中包含了用户的accessKey和secretKey信息  
        COSCredentials cred = new BasicCOSCredentials(accessKey, secretKey);  
        // 创建一个ClientConfig实例，其中包含了客户端的配置信息，例如超时时间、连接池大小等，这里设置了bucket所在的区域信息，region参数就是存储桶所在的区域简写信息  
        ClientConfig clientConfig = new ClientConfig(new Region(region));  
        // 创建一个COSClient实例，传入前面创建的cred和clientConfig作为参数，返回一个COSClient对象，该对象用于和COS服务进行交互  
        return new COSClient(cred, clientConfig);  
    }  
}
```
配置和初始化一个与腾讯云对象存储（COS）交互的客户端
