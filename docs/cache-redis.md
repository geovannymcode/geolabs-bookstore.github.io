## Uso de Spring Cache con Redis
Mejora el rendimiento de tu aplicación utilizando Spring Cache con Redis:

1. Configurar Redis en tu proyecto Spring Boot:
* Agrega la dependencia de Redis en tu pom.xml:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2. Configurar RedisTemplate y CacheManager:
```java
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {
    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(new RedisStandaloneConfiguration("server", 6379));
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory());
        return template;
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
        return RedisCacheManager.builder(connectionFactory).cacheDefaults(config).build();
    }
}
```
3. Usar caché en tus servicios:

```java 
@Service
public class UserService {
    @Cacheable(value = "users", key = "#userId")
    public User getUserById(Long userId) {
        // Lógica para obtener el usuario
    }
}
```

