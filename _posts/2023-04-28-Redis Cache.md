---
categories: Project Redis
tags: [spring, Redis, Cache, Chat]
---

# Redis란
Redis(Remote Dictionary Server)는 메모리(In-Memory)에 데이터를 저장하는 NoSQL 데이터 저장소이다.

DB는 데이터를 디스크에 저장하지만 Redis는 메모리에 데이터를 저장하기 때문에 데이터 조회와 저장 속도가 매우 빠르다.

이러한 특성 때문에 빠른 응답 속도가 필요한 서비스 환경에서 많이 사용된다.

Redis 데이터 구조

- String           
- List         
- Hash         
- Set         
- Sorted Set         

<br><br><br><br>

## Redis Cache
캐시(Cache)란 자주 사용되는 데이터를 임시로 저장해 두었다가 재사용하는 기술을 의미한다.

일반적으로 사용자가 요청할 때마다 데이터베이스에 접근하면 조회 시간이 오래 걸리고 DB 부하도 증가하게 된다.

하지만 캐시를 사용하면 자주 조회되는 데이터를 메모리에 저장해두고 바로 조회할 수 있기 때문에 성능을 크게 향상시킬 수 있다.

<br>

Redis Cache는 이러한 캐시 기능을 Redis를 이용해 구현하는 방식이다.

Redis는 메모리 기반 저장소이기 때문에 데이터 조회 속도가 매우 빠르며

따라서 빠른 응답 속도가 필요한 서비스에서 캐시 저장소로 많이 사용된다.

<br>

Spring에서 Redis를 캐시로 사용하는 방법은 크게 두 가지로 나뉜다.

첫 번째는 Spring Cache 어노테이션을 사용하는 방식이다.

두 번째는 RedisTemplate을 사용해 Redis를 직접 제어하는 방식이다.

이 글에서는 두 가지 방법을 모두 사용해 Redis Cache를 적용하는 과정을 설명한다.

<br>

[spring.io - cache](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache)

<br><br><br><br>
  

## Cache 설정

### 1. 의존성 추가
build.gradle에 의존성 추가 

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-redis' // Redis 연동
implementation 'org.springframework.boot:spring-boot-starter-cache'      // Spring Cache 사용
```

<br><br>

### 2. 설정 파일 구성
`application.properties`에 추가  

```text
# Spring Cache를 Redis로 사용
spring.cache.type=redis

# Redis 서버 정보
spring.redis.host=host.docker.internal
spring.redis.port=6379

# null 값 캐싱 여부
spring.cache.redis.cache-null-values=false
```

`spring.cache.type`을 redis로 설정하면 Spring Cache가 Redis를 사용하도록 동작한다.

cache-null-values 옵션은 null 값을 캐시에 저장할지 여부를 결정한다.

<br><br>

### 3. Redis 설정 파일 작성
Redis 캐시 기능을 활성화하고 Redis 연결 정보를 설정하기 위해 RedisConfig 를 작성

```java
@Configuration
@EnableCaching // Spring Cache 기능 활성화
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String redisHost; // Redis 서버 주소

    @Value("${spring.redis.port}")
    private int redisPort; // Redis 서버 포트

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        // Redis 서버와의 연결을 관리하는 객체
        return new LettuceConnectionFactory(redisHost, redisPort);
    }

    @Bean
    public CacheManager cacheManager() {
        // Redis 캐시 기본 설정
        RedisCacheConfiguration cacheConfig =
                RedisCacheConfiguration.defaultCacheConfig()
                        // Redis key를 문자열로 직렬화
                        .serializeKeysWith(
                                RedisSerializationContext.SerializationPair.fromSerializer(
                                        new StringRedisSerializer()
                                )
                        )
                        // Redis value를 JSON 형태로 직렬화
                        .serializeValuesWith(
                                RedisSerializationContext.SerializationPair.fromSerializer(
                                        new GenericJackson2JsonRedisSerializer()
                                )
                        )
                        // 캐시 TTL을 30분으로 설정
                        .entryTtl(Duration.ofMinutes(30));

        // Redis 기반 CacheManager 생성
        return RedisCacheManager.builder(redisConnectionFactory())
                .cacheDefaults(cacheConfig)
                .build();
    }
}
```

`@EnableCaching`은 Spring에서 캐시 기능을 활성화하기 위해 반드시 필요하다.

RedisConnectionFactory는 Redis 서버와의 실제 연결을 담당한다.

CacheManager는 Spring Cache 어노테이션과 Redis를 연결해주는 핵심 역할을 한다.

serializeKeysWith는 Redis에 저장되는 key를 문자열 형태로 저장하기 위한 설정이다.

serializeValuesWith는 객체 데이터를 JSON 형태로 변환해 Redis에 저장하기 위한 설정이다.

entryTtl은 캐시에 저장된 데이터의 유효 시간을 의미한다.

<br>

이 CacheManager 설정은 Spring Cache 어노테이션을 사용할 때만 적용된다.

RedisTemplate을 사용할 경우에는 이 설정이 자동으로 적용되지 않는다.

<br><br>

### 4. 캐시 적용하기 : annotation 활용
Spring Cache annotation은 서비스 메서드 단위로 캐시를 적용할 수 있게 해준다.

`@Cacheable`은 메서드 실행 결과를 캐시에 저장한다.

같은 파라미터로 다시 호출되면 메서드 로직을 실행하지 않고 캐시에서 값을 반환한다.

<br>

`@CacheEvict`는 캐시에 저장된 데이터를 제거할 때 사용한다.

<br>

`@CachePut`은 메서드 실행 결과를 항상 캐시에 저장한다.

캐시 여부와 관계없이 메서드 로직이 실행된다.

```java
@Cacheable(
        key = "#nickname",              // 캐시 key
        value = "createRoom",           // 캐시 이름
        unless = "#result == null",     // null 반환 시 캐시 저장 X
        cacheManager = "cacheManager"   // 사용할 CacheManager
)
public ChatRoomDto createRoom(String nickname) {
    // 채팅방 생성 로직
}
```
<br>

객체 타입 파라미터를 사용할 경우 SpEL을 이용해 내부 필드에 접근한다.

```java
@Cacheable(
        key = "#chatMessage.sender",    // 객체 내부 필드 접근
        value = "roomId",
        unless = "#chatMessage.roomId == null"
)
public String findRoomId(ChatMessage chatMessage) {
    // roomId 조회 로직
}
```

`@Cacheable`은 캐시에 값이 존재하면 메서드 로직을 실행하지 않는다.

`@CachePut`은 항상 메서드를 실행해 캐시를 최신 상태로 유지한다.

`unless` 옵션을 사용하면 특정 조건에서 캐시 저장을 방지할 수 있다.

`value`는 Redis에서 캐시를 구분하는 이름이다.

`key`는 실제 Redis에 저장되는 key 값이다. 

<br><br>

#### 4.1 @Cacheable 사용 시 문제점
기존에 `@Cacheable`을 사용했을 때 값이 정상적으로 조회될 때도 있고 조회되지 않을 때도 있었다.

CacheManager 설정을 변경해도 문제가 완전히 해결되지 않았다.

`@CachePut`으로 변경해 테스트하자 정상적으로 값이 조회되기 시작했다.

<br>

`@Cacheable`은 메서드 실행 전에 캐시를 먼저 확인해서 

캐시에 null 값이 저장되어 있으면 이후에도 계속 null이 반환된다.

<br>

`@CachePut`은 메서드를 실행한 후 결과를 강제로 캐시에 저장하면서 캐시 값이 항상 갱신된다.

이를 해결하기 위해 null 값 자체를 캐시에 저장하지 않도록 설정했다.

```java
@Bean
public CacheManager cacheManager() {
    RedisCacheConfiguration cacheConfig =
            RedisCacheConfiguration.defaultCacheConfig()
                    .disableCachingNullValues() // null 값 캐싱 비활성화
                    .entryTtl(Duration.ofHours(3)) // TTL 3시간
                    .serializeKeysWith(
                            RedisSerializationContext.SerializationPair.fromSerializer(
                                    new StringRedisSerializer()
                            )
                    )
                    .serializeValuesWith(
                            RedisSerializationContext.SerializationPair.fromSerializer(
                                    new GenericJackson2JsonRedisSerializer()
                            )
                    );

    return RedisCacheManager.builder(redisConnectionFactory())
            .cacheDefaults(cacheConfig)
            .build();
}
```

<br><br>

### 5. RedisTemplate을 활용한 Cache 적용
RedisTemplate은 Redis를 직접 제어할 수 있다.

Spring Cache 어노테이션이 메서드 반환값을 자동으로 캐싱하는 방식이라면

RedisTemplate은 key, value, TTL을 직접 지정해 저장한다.

반환값이 아닌 특정 데이터를 Redis에 저장해야 할 경우 RedisTemplate 방식이 적합하다.

<br><br><br>

#### 5.1 RedisTemplate 설정
RedisTemplate을 사용하기 위해 RedisConfig에 Bean 설정을 추가한다.

```java
@Configuration
@EnableCaching // Spring Cache 기능 활성화
public class RedisConfig {

    @Bean // Redis 데이터를 쉽게 다루기 위한 RedisTemplate Bean 등록
    public RedisTemplate<String, String> redisTemplate() {
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();

        // Redis 서버와의 연결 설정
        redisTemplate.setConnectionFactory(redisConnectionFactory());

        // key를 문자열로 직렬화
        redisTemplate.setKeySerializer(new StringRedisSerializer());

        // value를 문자열로 직렬화
        redisTemplate.setValueSerializer(new StringRedisSerializer());

        return redisTemplate;
    }

    @Bean // ChatRoomDto 객체를 Redis에 저장하기 위한 RedisTemplate
    public RedisTemplate<String, ChatRoomDto> chatRoomDtoRedisTemplate() {
        RedisTemplate<String, ChatRoomDto> redisTemplate = new RedisTemplate<>();

        // Redis 서버와의 연결 설정
        redisTemplate.setConnectionFactory(redisConnectionFactory());

        // key는 문자열로 저장
        redisTemplate.setKeySerializer(new StringRedisSerializer());

        // value는 JSON 형태로 직렬화
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());

        return redisTemplate;
    }
}
```

RedisTemplate은 제네릭 타입을 통해 key와 value 타입을 지정한다.

StringRedisSerializer는 문자열 형태로 데이터를 저장한다.

GenericJackson2JsonRedisSerializer는 객체 데이터를 JSON 형태로 변환해 저장한다.


<br><br><br><br>

## 결론
RedisTemplate을 사용하면 캐시를 저장할 때 TTL과 같이 직접 설정이 가능하다.

`redisTemplate.opsForValue().set(key, value);`

<br>

`@Cacheable`과 같이 어노테이션을 사용할 때는 메서드의 반환 값이 자동으로 캐시에 저장된다.    

어노테이션을 사용하면 무조건 반환되는 타입으로만 Redis에 저장이 되므로

반환되는 값이 아닌 데이터를 Redis에 저장할 때에는 직접 RedisTemplate를 통해 구현하면 된다.

<br><br><br><br>

**REFERENCE**                     
- [Spring Data Redis](https://docs.spring.io/spring-data/redis/reference/index.html)             
- [[Spring] Spring에서 Redis로 Cache 사용하기 (CrudRepository, RedisTemplate)](https://loosie.tistory.com/807)       
