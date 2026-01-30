---
name: redis-patterns
description: Redis patterns for caching, sessions, queues, and data structures. Use when working with Redis.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Redis Patterns

## Spring Redis Configuration

```java
@Configuration
@EnableCaching
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
    
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))
            .disableCachingNullValues()
            .serializeValuesWith(
                SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
        
        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .withCacheConfiguration("users", 
                config.entryTtl(Duration.ofMinutes(30)))
            .withCacheConfiguration("sessions", 
                config.entryTtl(Duration.ofHours(24)))
            .build();
    }
}
```

## Cache Annotations

```java
@Service
@RequiredArgsConstructor
public class UserService {
    
    @Cacheable(value = "users", key = "#id")
    public UserDto findById(UUID id) {
        return userRepository.findById(id)
            .map(userMapper::toDto)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    @CachePut(value = "users", key = "#result.id")
    @Transactional
    public UserDto update(UUID id, UserUpdateDto dto) {
        // update logic
    }
    
    @CacheEvict(value = "users", key = "#id")
    @Transactional
    public void delete(UUID id) {
        userRepository.deleteById(id);
    }
    
    @CacheEvict(value = "users", allEntries = true)
    public void clearAllCache() {
        log.info("Clearing all user cache");
    }
}
```

## Data Structures

```java
@Service
@RequiredArgsConstructor
public class RedisDataService {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    // String (simple cache)
    public void cacheValue(String key, Object value, Duration ttl) {
        redisTemplate.opsForValue().set(key, value, ttl);
    }
    
    public Object getValue(String key) {
        return redisTemplate.opsForValue().get(key);
    }
    
    // List (queue)
    public void enqueue(String queue, Object item) {
        redisTemplate.opsForList().leftPush(queue, item);
    }
    
    public Object dequeue(String queue) {
        return redisTemplate.opsForList().rightPop(queue);
    }
    
    // Set (unique items)
    public void addToSet(String key, Object... values) {
        redisTemplate.opsForSet().add(key, values);
    }
    
    public Set<Object> getSetMembers(String key) {
        return redisTemplate.opsForSet().members(key);
    }
    
    // Hash (object fields)
    public void setHashField(String key, String field, Object value) {
        redisTemplate.opsForHash().put(key, field, value);
    }
    
    public Map<Object, Object> getHash(String key) {
        return redisTemplate.opsForHash().entries(key);
    }
    
    // Sorted Set (leaderboard)
    public void addToLeaderboard(String key, String member, double score) {
        redisTemplate.opsForZSet().add(key, member, score);
    }
    
    public Set<Object> getTopN(String key, int n) {
        return redisTemplate.opsForZSet().reverseRange(key, 0, n - 1);
    }
}
```

## Node.js with ioredis

```ts
import Redis from 'ioredis';

const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: parseInt(process.env.REDIS_PORT || '6379'),
  password: process.env.REDIS_PASSWORD,
  retryStrategy: (times) => Math.min(times * 50, 2000),
});

// Cache-aside pattern
async function getUser(id: string): Promise<User | null> {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);
  
  const user = await db.findUser(id);
  if (user) {
    await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
  }
  return user;
}

// Distributed lock
async function acquireLock(key: string, ttlMs: number): Promise<boolean> {
  const result = await redis.set(`lock:${key}`, '1', 'PX', ttlMs, 'NX');
  return result === 'OK';
}

async function releaseLock(key: string): Promise<void> {
  await redis.del(`lock:${key}`);
}

// Rate limiting
async function checkRateLimit(userId: string, limit: number, windowSec: number): Promise<boolean> {
  const key = `ratelimit:${userId}`;
  const current = await redis.incr(key);
  if (current === 1) {
    await redis.expire(key, windowSec);
  }
  return current <= limit;
}
```

## Common Patterns

| Pattern | Data Structure | Use Case |
|---------|---------------|----------|
| Cache | String | Object caching with TTL |
| Session | Hash | User session data |
| Queue | List | Job processing |
| Unique Items | Set | Tags, followers |
| Leaderboard | Sorted Set | Rankings, scores |
| Rate Limit | String + INCR | API throttling |
| Pub/Sub | Channels | Real-time messaging |

## Best Practices

- Always set TTL on cached data
- Use appropriate data structures for your use case
- Implement cache invalidation strategy
- Use connection pooling
- Monitor memory usage with `INFO memory`
- Use `SCAN` instead of `KEYS` in production
- Consider Redis Cluster for high availability
