# Redis y Patrones de Caching

## Dependencias

```xml
<!-- Maven -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

## Configuración

```yaml
# application.yml
spring:
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:}
      timeout: 2000ms
  cache:
    type: redis
    redis:
      time-to-live: 3600000  # 1 hora
      cache-null-values: false
```

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
        
        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .withCacheConfiguration("users", 
                config.entryTtl(Duration.ofMinutes(30)))
            .withCacheConfiguration("products", 
                config.entryTtl(Duration.ofHours(6)))
            .build();
    }
}
```

## Anotaciones de Cache

### @Cacheable
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class UserService {
    
    private final UserRepository userRepository;
    
    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        log.info("Cache MISS - Loading user from DB: {}", id);
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    @Cacheable(value = "users", key = "#email", unless = "#result == null")
    public User findByEmail(String email) {
        return userRepository.findByEmail(email).orElse(null);
    }
}
```

### @CacheEvict
```java
@CacheEvict(value = "users", key = "#user.id")
public User update(User user) {
    return userRepository.save(user);
}

@CacheEvict(value = "users", allEntries = true)
public void clearUserCache() {
    log.info("User cache cleared");
}
```

### @CachePut
```java
@CachePut(value = "users", key = "#result.id")
public User create(CreateUserRequest request) {
    User user = userMapper.toEntity(request);
    return userRepository.save(user);
}
```

### Múltiples operaciones
```java
@Caching(
    evict = {
        @CacheEvict(value = "users", key = "#id"),
        @CacheEvict(value = "usersByEmail", key = "#user.email")
    }
)
public void delete(Long id) {
    userRepository.deleteById(id);
}
```

## Cache-Aside Pattern (Manual)

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class ProductService {
    
    private final ProductRepository productRepository;
    private final RedisTemplate<String, Product> redisTemplate;
    
    private static final String CACHE_PREFIX = "product:";
    private static final Duration TTL = Duration.ofHours(1);
    
    public Product findById(Long id) {
        String key = CACHE_PREFIX + id;
        
        // 1. Check cache
        Product cached = redisTemplate.opsForValue().get(key);
        if (cached != null) {
            log.debug("Cache HIT: {}", key);
            return cached;
        }
        
        // 2. Cache miss - fetch from DB
        log.debug("Cache MISS: {}", key);
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
        
        // 3. Update cache
        redisTemplate.opsForValue().set(key, product, TTL);
        
        return product;
    }
    
    public void invalidate(Long id) {
        String key = CACHE_PREFIX + id;
        redisTemplate.delete(key);
        log.info("Cache invalidated: {}", key);
    }
}
```

## Write-Through Pattern

```java
@Transactional
public Product save(Product product) {
    // 1. Save to DB
    Product saved = productRepository.save(product);
    
    // 2. Update cache
    String key = CACHE_PREFIX + saved.getId();
    redisTemplate.opsForValue().set(key, saved, TTL);
    
    return saved;
}
```

## Distributed Lock con Redis

```java
@Service
@RequiredArgsConstructor
public class DistributedLockService {
    
    private final StringRedisTemplate redisTemplate;
    
    public boolean tryLock(String key, Duration timeout) {
        String lockKey = "lock:" + key;
        Boolean acquired = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, "locked", timeout);
        return Boolean.TRUE.equals(acquired);
    }
    
    public void unlock(String key) {
        String lockKey = "lock:" + key;
        redisTemplate.delete(lockKey);
    }
    
    public <T> T executeWithLock(String key, Duration timeout, Supplier<T> action) {
        if (!tryLock(key, timeout)) {
            throw new LockAcquisitionException("Could not acquire lock: " + key);
        }
        try {
            return action.get();
        } finally {
            unlock(key);
        }
    }
}

// Uso
public void processOrder(Long orderId) {
    lockService.executeWithLock("order:" + orderId, Duration.ofMinutes(5), () -> {
        // Proceso exclusivo
        return orderService.process(orderId);
    });
}
```

## Rate Limiting con Redis

```java
@Service
@RequiredArgsConstructor
public class RateLimiter {
    
    private final StringRedisTemplate redisTemplate;
    
    public boolean isAllowed(String key, int maxRequests, Duration window) {
        String redisKey = "ratelimit:" + key;
        Long currentCount = redisTemplate.opsForValue().increment(redisKey);
        
        if (currentCount == 1) {
            redisTemplate.expire(redisKey, window);
        }
        
        return currentCount <= maxRequests;
    }
}

// Uso en Controller
@GetMapping("/api/resource")
public ResponseEntity<?> getResource(HttpServletRequest request) {
    String clientIp = request.getRemoteAddr();
    
    if (!rateLimiter.isAllowed(clientIp, 100, Duration.ofMinutes(1))) {
        return ResponseEntity.status(429).body("Rate limit exceeded");
    }
    
    return ResponseEntity.ok(service.getData());
}
```

## Mejores Prácticas

1. **TTL apropiado** - Datos que cambian poco = TTL largo
2. **Keys con prefijo** - `user:123`, `product:456`
3. **Evitar cache stampede** - Usar locks o TTL aleatorio
4. **Serialización eficiente** - JSON o MessagePack
5. **Monitorear hit ratio** - Objetivo > 80%
6. **Invalidar explícitamente** - No depender solo de TTL
7. **Fallback a DB** - Si Redis falla, funcionar sin cache

## Comandos Redis CLI

```bash
# Conectar
redis-cli -h localhost -p 6379

# Ver keys
KEYS user:*

# Ver TTL
TTL user:123

# Ver valor
GET user:123

# Limpiar cache
DEL user:123
FLUSHDB  # ¡Cuidado! Borra todo
```
