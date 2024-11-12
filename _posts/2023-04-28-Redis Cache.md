---
categories: Project Redis
tags: [spring, Redis, Cache, Chat]
---

# Redis란
Redis(Remote Dictionary System)는 인메모리 데이터 저장소로 

주로 세션 관리, 캐싱, db 및 메시지 브로커 역할을 수행하는 오픈 소스다.

다양한 데이터 타입(String, List, Hash, Set 등)을 지원하며 

단일 스레드로 동작하여 동시성 이슈가 적고 빠른 속도를 제공한다. 

유연한 데이터 모델 지원하여 대용량 데이터를 처리하는 데 유용한 NoSQL 데이터베이스로 활용된다.      

<br><br><br><br>

## Redis Cache
캐싱(Cache)은 자주 참조되는 데이터를 메모리에 저장하여 반복 작업을 줄이고 성능을 향상시키는 것을 의미한다. 

Redis Cache는 인메모리 데이터 저장소로, 빠른 응답이 필요한 애플리케이션에서 효과적이다. 

Redis Cache를 적용하는 방법은 크게 두 가지로 나눌 수 있다.

<br>

**1.** Spring 어노테이션 사용 (`@Cacheable`, `@CachePut` 등)  

**2.** RedisTemplate 사용

이 글에서는 두 가지 방법을 통해 Redis Cache를 적용하는 과정을 설명한다.

<br><br>   

나는 두 가지 방법 모두 사용하여 Redis Cache를 적용했다.

RedisTemplate을 사용하여 특정 데이터를 선택적으로 저장하고 검색했으며

Annotation을 사용하여 전체 method의 결과를 캐시했다.

<br>

[spring.io - cache](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache)

<br><br><br><br>  

## Cache 설정   
### 1. 의존성 추가
build.gradle에 의존성 추가  
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
implementation 'org.springframework.boot:spring-boot-starter-cache'
```

<br><br><br>

### 2. 설정 파일 구성
`application.properties`에서 Redis 정보 추가
```
#redis cache
spring.cache.type = redis

# redis docker
spring.redis.host = host.docker.internal
spring.redis.port = 6379
spring.cache.redis.cache-null-values=false  # null 값도 캐시할지 여부
```

<br><br><br>   

### 3. Redis 설정 파일 작성

Redis Cache 기능을 활성화하고 Redis 연결 정보를 설정하기 위해 RedisConfig 클래스를 작성했다.

```java
@Configuration
@EnableCaching // 캐시 기능 활성화 
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(redisHost, redisPort);
    }

    @Bean
    public CacheManager cacheManager() { // Spring 어노테이션 사용할 때 작성하는 설정 
        RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig()
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
            .entryTtl(Duration.ofMinutes(30));
        return RedisCacheManager.builder(redisConnectionFactory())
            .cacheDefaults(cacheConfig)
            .build();
    }
}
```

- `serializeKeysWith()`: Redis에 저장되는 key의 직렬화 방식을 설정.        
    `StringRedisSerializer`를 사용해 문자열로 직렬화

- `serializeValuesWith()` : Redis에 저장되는 value의 직렬화 방식을 설정.       
     `GenericJackson2JsonRedisSerializer`를 사용해 JSON 형식으로 직렬화하여 객체 데이터를 JSON으로 저장

- `entryTtl` : 캐시 데이터의 유효 시간 설정 → 30분 

<br>

참고)

CacheManager 설정은 Spring 어노테이션 기반 캐시와 연동되므로 

RedisTemplate으로 Redis를 제어할 때는 이 설정을 따르지 않는다. 

RedisTemplate에서는 TTL 등 설정을 개별적으로 코드 내에서 정의할 수 있다. 

<br><br><br>

### 4. 캐시 적용하기 : annotation 활용
사용할 서비스 method에 annotation 적용하면 된다.

- `@Cacheable` : 메서드의 결과를 캐시에 저장하고, 이후에 동일한 인자로 메서드가 호출될 때 캐시에서 결과를 반환
  
  - 캐시에 데이터가 존재하면 메서드의 로직을 실행하지 않고 캐시에서 데이터를 조회하여 반환
  - 캐시에 데이터가 없으면 메서드의 로직을 실행하고 그 결과를 캐시에 저장한 후 반환

<br>

- `@CacheEvict` : 지정된 캐시에서 데이터를 제거. 주로 데이터를 삭제할 때 사용

<br>

- `@CachePut` : 메서드의 결과를 강제로 캐시에 저장. 주로 데이터를 갱신할 때 사용
  - 메서드의 로직을 실행하고 그 결과를 캐시에 저장
  - 항상 메서드의 로직을 실행하므로 캐시에서 데이터를 조회하지 않는다.  

<br>
 
condition, unless 어노테이션 옵션으로 특정 조건에 따른 캐시적용여부 설정도 가능하다.

<br><br>

```java
@Cacheable(key = "#nickname", value = "createRoom", unless = "#result == null", cacheManager = "cacheManager")
public ChatRoomDto createRoom(String nickname) {
    // 메서드 로직
}
```
`@Cacheable`은 캐시에 이미 데이터가 있으면 메서드 로직을 실행하지 않고 캐시에서 데이터를 반환한다. 

`@CachePut`을 사용하면 매번 메서드 로직이 실행되어 최신 상태가 유지된다.
  
- `unless = "#result == null"` : 반환값이 null일 경우 캐시 저장을 하지 않도록 설정
                                   
- `cacheManager = "cacheManager"` : 위의 config에서 작성한 cacheManager 사용

<br><br>     

`@Cacheable`의 key 속성은 캐시할 데이터의 key를 지정하고, value 속성은 캐시할 데이터의 value를 지정한다.      

여기서 key는 사용자의 nickname이 되고 value는 `createRoom()`의 반환값이 저장 될 것이다.

`@Cacheable` 어노테이션에서 value를 설정할 때 mehtod의 반환 타입이 void인 경우 

value를 지정할 수 없으므로 value = ""와 같이 빈 문자열로 설정해야 한다.

<br><br>     

#### 4.1 문제점  
기존에 `@Caceable`을 사용했으나 값을 제대로 가져올 때가 있고 없을 때가 있었다.

처음엔 cacheManager 설정도 바꿔보다가 

`@CachePut`으로 바꾸고 test를 해보니 값을 제대로 가져오기 시작했다.

왜 `@Caceable`에서 `@CachePut`로 수정을 하니 제대로 동작했을까?

<br><br> 

`@Cacheable` 어노테이션은 메서드가 호출되기 전에 캐시를 확인하고, 캐시에 저장된 결과가 있으면 해당 결과를 반환한다. 

만약 캐시에 값이 없을 때 null을 반환한다면, 이후에도 항상 null을 반환할 것이다.

<br> 

`@CachePut` 어노테이션은 메서드의 로직을 실행하고 그 결과를 강제로 캐시에 저장한다. 

매번 메서드가 호출될 때마다 로직이 실행되고 그 결과가 캐시에 갱신된다. 

<br>

그래서 null 값을 방지 하기 위해 `unless = "#result == null"`로 작성하고 RedisConfig에서도 추가했다.

```java
@Bean
public CacheManager cacheManager() {
    RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig()
            .disableCachingNullValues() // null value cache X
            .entryTtl(Duration.ofHours(3))
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
    return RedisCacheManager.builder(redisConnectionFactory())
            .cacheDefaults(cacheConfig) // 기본 캐시에 TTL 적용
            .build();
}
```

<br><br><br>

### 5. TTL 설정 
TTL은 "Time To Live"의 약자로, 캐시된 데이터가 유효한 시간을 의미한다.

Redis에서 기본 TTL을 설정하거나, 특정 캐시별로 TTL을 다르게 적용할 수도 있다.

<br>

데이터가 없는 경우 캐시에 null 값을 저장할 경우에는 TTL이 적용되지 않는다.  

캐시된 데이터가 null 일 때도 TTL을 적용하도록 설정하고 싶다면

`RedisCacheConfiguration.defaultCacheConfig().disableCachingNullValues()`를 사용하면 된다.  

<br>

```java
@Bean
public CacheManager cacheManager() {
    RedisCacheConfiguration defaultCacheConfig = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(3)) // 기본 TTL은 3시간
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

    RedisCacheConfiguration roomIdCacheConfig = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(5)) // roomId 캐시의 TTL을 5시간으로 설정
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

    return RedisCacheManager.builder(redisConnectionFactory())
            .cacheDefaults(defaultCacheConfig)
            .withCacheConfiguration("roomId", roomIdCacheConfig) // ROOMID 캐시에 대한 설정 적용
            .build();
}
```

`.withCacheConfiguration("roomId", roomIdCacheConfig)` : roomId 캐시에 대한 설정을 cacheConfig로 지정

CacheManagerBuilder에서 roomId 캐시에 대한 설정을 지정하기 위한 것이다.   

어노테이션을 활용해서 Cache를 적용할 때 value 값으로 "roomId"를 설정하면

해당 method의 return값은 TTL이 5시간 적용되어 활용된다.  

<br>

여기서 roomId는 아래 코드 처럼 value에 적힌 값이 된다.

`@Cacheable(key = "#chatMessage.sender", value = "roomId", unless = "#chatMessage.roomId == null")`

<br><br>

만약 모든 어노테이션에 캐시를 적용할 것이라면 아래와 같이 작성하면 된다.

```java
@Bean
public CacheManager cacheManager() {
    RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofHours(3))
        .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
    return RedisCacheManager.builder(redisConnectionFactory())
        .cacheDefaults(cacheConfig) // 기본 캐시에 TTL 적용
        .build();
}   
```

<br><br><br><br>

### 6. RedisTemplate을 활용한 Cache 적용  
RedisTemplate을 사용하면 데이터 접근과 제어를 세부적으로 설정할 수 있다. 

<br>

#### 6.1 RedisTemplate 설정 
```java
@Configuration
@EnableCaching
public class RedisConfig {
    @Bean //Redis 데이터에 쉽게 접근하기 위한 코드
    public RedisTemplate<String, String> redisTemplate() { //RedisTemplate 에 LettuceConnectionFactory 을 적용
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        return redisTemplate;
    }

    @Bean
    public RedisTemplate<String, ChatRoomDto> chatRoomDtoRedisTemplate() {
        RedisTemplate<String, ChatRoomDto> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer()); // JSON 직렬화 설정
        return redisTemplate;
    }
}
```

<br><br><br>

#### 6.2 데이터 저장과 조회                          
ChatMessage.sender와 nickname은 같은 값              

```java
private final RedisTemplate<String, String> redisTemplate;

public void addRedis(ChatMessage chatMessage){
    long expireTimeInSeconds = 24 * 60 * 60;
    long creationTimeInMillis = System.currentTimeMillis();
    long remainingTimeInSeconds = expireTimeInSeconds - ((System.currentTimeMillis() - creationTimeInMillis) / 1000);
    redisTemplate.opsForValue().set(chatMessage.getSender(), chatMessage.getRoomId(), remainingTimeInSeconds, TimeUnit.SECONDS);
}

public String getRedis(String nickname){
    return redisTemplate.opsForValue().get("roomId:" + nickname);
}
```
redisTemplate의 `opsForValue()` 를 이용해 `chatMessage.getSender()`를 key로 하고 

`chatMessage.getRoomId()`를 value로 하는 key-value 쌍을 Redis에 저장하고 있다. 

Redis에 저장되는 데이터의 구조는 sender가 key이고 roomId가 value인 형태가 된다.

<br><br>

`redisTemplate.opsForValue().set()`의 마지막 인자로 TTL을 지정하는 방식으로 Redis 캐시에 TTL을 설정했다. 

24시간을 기준으로 현재시간으로 부터 남은 시간을 초 단위로 계산해서 만료시간 설정한다.

<br><br>

어노테이션을 사용하면 bean에서 설정한 TTL이 적용되고

RedisTemplate을 사용해 데이터를 저장하면 코드에서 설정한 TTL(24시간)이 적용된다.

<br><br>

---

## 💡 결론
RedisTemplate을 사용하면 캐시를 저장할 때 TTL과 같이 직접 설정이 가능하다.

`redisTemplate.opsForValue().set(key, value);`

<br>

`@Cacheable과`과 같이 어노테이션을 사용할 때는 메서드의 반환 값이 자동으로 캐시에 저장된다.

어노테이션을 사용하면 무조건 반환되는 타입으로만 Redis에 저장이 되므로, 

반환되는 값이 아닌 데이터를 Redis에 저장할 때에는 직접 RedisTemplate를 통해 구현하면 된다.

<br><br><br><br>

**REFERENCE**           
- [Spring Cache, 제대로 사용하기](https://gngsn.tistory.com/157)                   
- [[JAVA Spring Boot] Rest API + 레디스 캐시 (Redis Cache) 적용 및 샘플 예제](https://kim-oriental.tistory.com/28)            
- [SpringBoot기반 Redis Cache 활용법](https://yonguri.tistory.com/82)                   
- [Cache 기능 Redis로 구현하기](https://pamyferret.tistory.com/25)   
- [[Spring] Spring에서 Redis로 Cache 사용하기 (CrudRepository, RedisTemplate)](https://loosie.tistory.com/807)       
- [Springboot: redis를 통해 캐시 기능 간단 적용](https://khdscor.tistory.com/m/99)      
