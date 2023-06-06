---
categories: Project
tags: [spring, Redis, Cache]
---

# Redis란
Redis는 인메모리 데이터 저장소로, 다양한 데이터 구조를 지원하며, 빠른 속도와 Pub/Sub 메시징 시스템을 제공한다.

주로 세션 관리와 대용량 데이터 처리에 사용되며, 캐시, 데이터베이스, 메시지 브로커 등으로 활용된다. 

<br>

Redis는 다양한 데이터 구조를 지원하며

키-값 저장소로 문자열, 해시, 리스트, 세트, 정렬된 세트 등 다양한 데이터 타입을 지원한다.

<br>

Redis는 오픈 소스로, Single Thread로 동작하기 때문에 멀티스레드 환경에서의 동시성 이슈를 걱정할 필요가 없으며, 

세션 관리를 위해 사용되면 빠른 성능과 서버 부하 감소를 제공한다. 

<br>

또한, Redis는 유연한 데이터 모델을 지원하여 대용량 데이터 처리에 유용한 NoSQL 데이터베이스다.         

<br><br><br>

## Redis Cache
*Cache란 자주 참조되는 데이터를 메모리에 저장해, 반복적인 작업을 최소화하여 시스템 성능을 향상시키는 것을 의미

Redis Cache는 Redis를 사용한 메모리 기반 데이터 저장소의 캐싱 방법이다.

이를 통해 자주 참조되는 데이터를 메모리에 저장하여 반복 작업을 최소화하고 시스템 성능을 향상시킬 수 있다.

Redis Cache는 인-메모리 데이터 저장소로써, 빠른 응답 시간을 요구하는 애플리케이션에서 데이터를 캐싱하는 데 사용된다. 

<br>

Redis 캐시는 캐시 된 데이터를 직렬화하여 저장하기 때문에, 

API의 응답값 데이터 타입이 String 타입이 아닌 경우 Serializer 에러가 발생한다. 

거의 대부분의 API 서비스의 응답은 json 형태로 이루어지므로, 

Json 데이터를 직렬화하는 Serializer로 **Jackson2JsonRedisSerializer**나 

**GenericJackson2JsonRedisSerializer**를 사용할 수 있다.

<br>

Redis 캐시를 사용하려면 Redis 서버와의 연결을 설정해야 한다. 

이를 위해서는 **RedisConnectionFactory**와 **RedisTemplate**을 구성해야 한다. 

RedisTemplate을 구성할 때, Key Serializer와 Value Serializer를 구성해야 하며, 

적절한 Serializer를 구성하지 않으면 Redis에서는 객체를 일반적으로 저장할 수 없다. 

기본적으로 StringRedisSerializer를 사용하여 key와 value를 저장하도록 구성할 수 있다. 

하지만 경우에 따라 사용자 정의 Serializer가 필요할 수 있다.

<br>

[spring.io - cache](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache)

<br><br>

### build.gradle
```
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

<br><br>

### application.properties
```
#redis cache
spring.cache.type = redis
spring.cache.redis.cache-null-values=true

# redis docker
spring.redis.host = host.docker.internal
spring.redis.port = 6379
```

<br><br>

### XXXApplication
```java
@SpringBootApplication
@EnableCaching // cache 사용
public class AdmeApplication { 
    public static void main(String[] args) {
        SpringApplication.run(AdmeApplication.class, args);
    }
}
```
<br><br>



<br><br>

### Config

Redis 캐싱 기능을 사용하기 위해 Redis 설정을 해주어야 한다.

`@EnableCaching` 어노테이션을 추가하여 캐시 기능을 사용하도록 설정한다. 

또한 Redis 연결 정보를 설정해야 하기 때문에 RedisConfig 클래스를 생성한다.
```java
@Configuration
@EnableCaching
public class RedisConfig {
    @Value("${spring.redis.host}")
    private String redisHost;
    @Value("${spring.redis.port}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(redisHost, redisPort);
    }

    @Bean //Redis 데이터에 쉽게 접근하기 위한 코드
    public RedisTemplate<?, ?> redisTemplate() { //RedisTemplate 에 LettuceConnectionFactory 을 적용
        RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        return redisTemplate;
    }

    @Bean
    public StringRedisTemplate stringRedisTemplate() { //StringRedisTemplate은 문자열을 다룰 때 사용
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        stringRedisTemplate.setKeySerializer(new StringRedisSerializer());
        stringRedisTemplate.setValueSerializer(new StringRedisSerializer());
        stringRedisTemplate.setConnectionFactory(redisConnectionFactory());
        return stringRedisTemplate;
    }
    
    @Bean
    public CacheManager cacheManager() {
        RedisCacheManager.RedisCacheManagerBuilder builder = RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(redisConnectionFactory());
        RedisCacheConfiguration configuration = RedisCacheConfiguration.defaultCacheConfig()
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))// Value Serializer 변경
            .entryTtl(Duration.ofMinutes(30));
    builder.cacheDefaults(configuration);
    return builder.build();
    }
}
```

코드를 하나씩 살펴본다.

<br><br><br>

### Serializer 설정 방법
```java
@Configuration
@EnableCaching
public class RedisConfig {
    @Value("${spring.redis.host}")
    private String redisHost;
    @Value("${spring.redis.port}")
    private int redisPort;

```
Redis 연결 정보를 설정하기 위한 `redisConnectionFactory()`와 Redis 데이터에 쉽게 접근하기 위한 `redisTemplate()`를 생성한다.  

<br><br>

`redisConnectionFactory()` 에서는 LettuceConnectionFactory 클래스를 이용하여 RedisConnectionFactory를 생성한다.

```java
@Bean
public RedisConnectionFactory redisConnectionFactory() {
    return new LettuceConnectionFactory(redisHost, redisPort);
}
```

<br><br><br>

#### 1. Key Serializer와 Value Serializer 설정
`redisTemplate()`에서는 RedisTemplate 클래스를 이용하여 Redis에 저장된 데이터를 쉽게 다룰 수 있는 redisTemplate을 생성한다.

```java
@Bean
public RedisTemplate<?, ?> redisTemplate() {
    RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(redisConnectionFactory());
    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.setValueSerializer(new StringRedisSerializer());
    return redisTemplate;
}
```
RedisTemplate을 구성할 때, 사용자 정의 Serializer를 정의하지 않으면 Redis에서는 객체를 일반적으로 저장할 수 없다. 

따라서 RedisTemplate 구성에서 Key Serializer와 Value Serializer를 구성하는 것이 중요하다.

위의 코드에서는 **StringRedisSerializer**를 사용하여 key와 value를 저장하도록 구성했지만,

경우에 따라 사용자 정의 Serializer가 필요할 수 있다.

<br><br>

StringRedisTemplate은 문자열을 다룰 때 사용하며 `redisTemplate()`와 유사하게 구현된다.
```java
@Bean
public StringRedisTemplate stringRedisTemplate() {
    StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
    stringRedisTemplate.setKeySerializer(new StringRedisSerializer());
    stringRedisTemplate.setValueSerializer(new StringRedisSerializer());
    stringRedisTemplate.setConnectionFactory(redisConnectionFactory());
    return stringRedisTemplate;
}
```

<br><br><br>

#### 2. RedisCacheConfiguration으로 설정 객체 생성
`cacheManager()` 에서는 Redis 캐싱 기능을 구현하기 위한 설정을 해준다.

```java
@Bean
public CacheManager cacheManager() {
    RedisCacheManager.RedisCacheManagerBuilder builder = RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(redisConnectionFactory());
    RedisCacheConfiguration configuration = RedisCacheConfiguration.defaultCacheConfig()
        .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))// Value Serializer 변경
        .entryTtl(Duration.ofMinutes(30));
    builder.cacheDefaults(configuration);
    return builder.build();
}
```

Redis 캐싱 기능을 사용하기 위해서는 **RedisCacheConfiguration**을 이용하여 설정 객체를 생성해야 한다. 

이 객체는 `defaultCacheConfig()`를 이용하여 생성하며, key 값과 value 값을 어떻게 직렬화할지를 정의한다. 

직렬화 방법을 정의하지 않으면 캐시 데이터를 저장할 수 없는 에러가 발생할 수 있다.

<br>

`serializeKeysWith()`: key 값을 어떻게 직렬화할지 정의한다. 

`RedisSerializationContext` : 직렬화할 수 있는 방법을 제공하는 인터페이스를 이용한다. 

`RedisSerializationContext.SerializationPair.fromSerializer()` : 직렬화에 사용할 어댑터를 함수 안에 넣은 타입에 맞게 반환하는 static 함수

<br>

예를 들어, **StringRedisSerializer**를 이용하여 key 값을 직렬화한다면, 

key String 값을 byte로 변환하여 저장하고, 가져올 때는 다시 변환하여 가져온다.

<br>

`GenericJackson2JsonRedisSerializer`는 캐시 데이터(Object)를 JSON으로 직렬화하여 Redis에 저장하고, 

가져올 때는 JSON을 Object로 변환하여 가져온다. 

이 방식을 사용하면 객체를 그대로 Redis에 저장할 수 있다.

<br>

RedisCacheManagerBuilder를 이용하여 RedisCacheManager를 생성하고, 

RedisCacheManagerBuilder의 `cacheDefaults()` 를 이용하여 설정 객체를 설정한다. 

마지막으로 RedisCacheManager를 반환하여 캐시 매니저를 생성한다. 

이때, `RedisConnectionFactory()`를 이용하여 Redis에 연결한다.

<br><br><br>

#### 3. Jackson2JsonRedisSerializer나 GenericJackson2JsonRedisSerializer 사용
Redis에서 key는 문자열 형태로, value는 일반적으로 문자열이 아닌 객체 형태로 저장된다. 

이때, 객체를 직렬화하여 저장해야 하는데, 

**Jackson2JsonRedisSerializer**와 **GenericJackson2JsonRedisSerializer**를 사용하여 JSON 형태로 직렬화할 수 있다.

<br>

**Jackson2JsonRedisSerializer**는 Jackson 라이브러리를 이용하여 객체를 JSON 형태로 직렬화하고 역직렬화할 수 있는 Serializer이다. 

**GenericJackson2JsonRedisSerializer**는 Jackson2JsonRedisSerializer와 동일하게 Jackson 라이브러리를 사용하지만, 

객체를 JSON 형태로 직렬화할 때 클래스 타입 정보도 함께 저장하여 역직렬화 시에도 클래스 정보를 유지할 수 있다는 장점이 있다.

```java
@Bean
public RedisTemplate<?, ?> redisTemplate() { 
    RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(redisConnectionFactory());
    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(Object.class));
    return redisTemplate;
}

```

Jackson2JsonRedisSerializer 대신 GenericJackson2JsonRedisSerializer를 사용하는 경우에는 아래와 같이 구성할 수 있다.
```java
@Bean
public RedisTemplate<?, ?> redisTemplate() { 
    RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(redisConnectionFactory());
    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
    return redisTemplate;
}
```

<br><br><br>

#### redisTemplate Bean 구성
RedisTemplate을 구성할 때, 사용자 정의 Serializer를 정의하지 않으면 Redis에서는 객체를 일반적으로 저장할 수 없다. 

따라서 RedisTemplate 구성에서 Key Serializer와 Value Serializer를 구성하는 것이 중요하다. 

코드에서는 StringRedisSerializer를 사용하여 key와 value를 저장하도록 구성했지만, 

경우에 따라 사용자 정의 Serializer가 필요할 수 있다.

ex)      
```java
@Bean
public RedisTemplate<String, ChatRoomDto> redisTemplate(RedisConnectionFactory connectionFactory) {
    RedisTemplate<String, ChatRoomDto> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(connectionFactory);
    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(ChatRoomDto.class));
    return redisTemplate;
}
```

<br><br><br><br>

### 캐시 구성
Redis 캐시를 사용할 때 다양한 구성 옵션이 있을 수 있으며, 

TTL은 "Time To Live"의 약자로, 캐시된 데이터가 유효한 시간을 의미한다.

TTL(Time To Live)을 사용하면 캐시된 데이터가 일정 기간 이후에 자동으로 삭제되어 원본 데이터를 정확하게 반영할 수 있다.

그러나, 데이터가 없는 경우 캐시에 null 값을 저장할 경우에는 TTL이 적용되지 않기 때문에, 

이 경우에는 캐시된 데이터가 null 일 때도 TTL을 적용하도록 설정하는 것이 좋다. 

이를 위해서는 `RedisCacheConfiguration.defaultCacheConfig().disableCachingNullValues()`를 사용한다.  

<br>

또한, TTL을 일부 코드에만 적용할 수 있으므로 TTL을 사용할 코드와 사용하지 않을 코드를 구분하여 작성할 수 있다. 

아래는 TTL을 적용한 캐시와 TTL을 적용하지 않은 캐시를 따로 정의하고, 

RedisCacheManager를 이용해 roomId 캐시에는 TTL을 적용하고, 그 외 캐시에는 TTL을 적용하지 않도록 구성한 코드이다.  


<br><br>

**TTL을 일부 코드에만 적용할 수 있으므로 TTL을 사용할 코드와 사용하지 않을 코드를 구분해서 작성**

```java
@Bean
public CacheManager cacheManager() {
    long expireTimeInSeconds = 24 * 60 * 60;
    long creationTimeInMillis = System.currentTimeMillis();
    long remainingTimeInSeconds = expireTimeInSeconds - ((System.currentTimeMillis() - creationTimeInMillis) / 1000);
    RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofMinutes(30))
        .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
    RedisCacheConfiguration cacheConfigWithoutNullValues = RedisCacheConfiguration.defaultCacheConfig()
        .disableCachingNullValues()
        .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
    return RedisCacheManager.builder(redisConnectionFactory())
        .cacheDefaults(cacheConfigWithoutNullValues)
        .withCacheConfiguration("roomId", cacheConfig)
        .build();
}
```
위 코드에서는 `RedisCacheConfiguration.defaultCacheConfig().disableCachingNullValues()`를 사용한다.  

cacheConfigWithoutNullValues와 TTL을 적용한 cacheConfig를 따로 정의하고, 

RedisCacheManager.builder()를 이용해 roomId 캐시에는 cacheConfig를, 

그 외 캐시에는 cacheConfigWithoutNullValues를 적용하여 조합하도록 구성했다.

<br>

따라서, TTL을 일부 코드에만 적용할 수 있으므로 TTL을 사용할 코드와 사용하지 않을 코드를 구분하는 것이 가능하다. 

단, 여러 개의 캐시가 필요할 경우 이러한 방식을 사용하기보다는 

각각의 캐시에 대해 RedisCacheConfiguration을 따로 정의하여 구성하는 것이 더 명확하고 좋다.

<br>

`.withCacheConfiguration("roomId", cacheConfig)` : roomId 캐시에 대한 설정을 cacheConfig로 지정

CacheManagerBuilder에서 roomId 캐시에 대한 설정을 지정하기 위한 것이다.   

만약 TTL이 적용되어야 할 캐시에 대해 TTL을 적용하지 않은 경우, 해당 캐시는 기대한 대로 작동하지 않을 수 있다. 

따라서 TTL이 적용되어야 할 캐시에 대해서는 TTL을 꼭 적용하는 것이 좋다

<br><br>

**"roomId"??** (아래 code 예시 참고)

@Cacheable 어노테이션에서 value 속성으로 설정된 값과 일치해야한다. 

즉, value = "roomId"로 설정된 `@Cacheable` 어노테이션에서 사용하는 캐시 이름이 roomId이므로 

`withCacheConfiguration()`를 사용하여 해당 캐시에 대한 캐시 구성을 지정하는 것이다. 

Redis에 저장된 값은 chatMessage.getRoomId()로 지정된다.

<br><br><br>

#### stringRedisTemplate Bean 구성
Redis에서 문자열 데이터를 처리하기 위해 stringRedisTemplate Bean을 사용한다. 

RedisTemplate과 유사하게 Serializer가 필요하다. 

StringRedisSerializer를 사용하여 문자열 데이터를 저장하도록 구성했지만, 다른 Serializer가 필요한 경우가 있을 수 있다.

<br><br>

#### Redis 연결 구성
LettuceConnectionFactory를 사용하여 Redis 연결을 구성했다. 

그러나 사용하는 Redis 클라이언트 라이브러리나 Redis 클라우드 서비스에 따라 다른 연결 구성이 필요할 수 있다.


<br><br><br>

**주의할 점!**

아래 코드에 문제가 있다.  
```java
@Cacheable(key = "#chatMessage.sender", value = "roomId", unless = "#chatMessage.roomId == null")
public void addRedis(ChatMessage chatMessage){
    long expireTimeInSeconds = 24 * 60 * 60;
    long creationTimeInMillis = System.currentTimeMillis();
    long remainingTimeInSeconds = expireTimeInSeconds - ((System.currentTimeMillis() - creationTimeInMillis) / 1000);
    redisTemplate.opsForValue().set(chatMessage.getSender(), chatMessage.getRoomId(), remainingTimeInSeconds, TimeUnit.SECONDS);
    //redisTemplate의 opsForValue() 를 이용해 chatMessage.getSender()를 key로 하고 chatMessage.getRoomId()를 value로 하는 key-value 쌍을 Redis에 저장하고 있다. 
    // 그래서 Redis에 저장되는 데이터의 구조는 sender가 key이고 roomId가 value인 형태가 된다.
}
```
☑️ `@Cacheable` 어노테이션과 `redisTemplate`을 같이 사용하지 않는다.

redisTemplate은 Spring에서 제공하는 Redis와의 상호작용을 담당하는 클래스다.

redisTemplate을 사용하면 Redis에 데이터를 저장하고 검색할 수 있다. 

그러나 redisTemplate을 사용하여 저장하는 데이터는 일반적으로 휘발성인 메모리 캐시로 저장되며

서버가 다시 시작되면 데이터가 사라진다.

<br>

`@Cacheable`은 Spring의 캐싱 추상화를 통해 메모리나 영구적인 캐시 저장소에 데이터를 저장하고 관리한다. 

기본적으로 메모리 캐시를 사용하지만, 필요에 따라 Redis와 같은 외부 캐시 저장소를 사용할 수도 있다. 

`@Cacheable` 어노테이션을 사용하면 데이터를 캐싱하여 반복적인 연산을 피하고 성능을 향상시킬 수 있다.

그러므로 별도로 redisTemplate을 사용하여 값을 저장할 필요가 없다.

<br>

→ redisTemplate과 `@Cacheable`은 서로 다른 기능을 가지고 있다. 

redisTemplate은 Redis와의 상호작용을 위해 사용되고 

`@Cacheable`은 Spring의 캐싱 추상화를 활용하여 캐싱 기능을 제공한다.

<br>

따라서 해당 메소드를 사용하고 있는 service 로직에 어노테이션을 붙여준다.

<br><br>

***Controller에 작성하면 안되는 이유**

Controller는 사용자의 요청을 처리하고 응답을 생성하는 역할을 수행한다.

Controller에서 DB 조회를 직접 수행하고 결과를 cache하게 되면

- 역할과 책임의 분리 : DB 조회와 같은 비즈니스 로직은 Service 계층에 위치하는 것이 좋다.
- 재사용성 : Service 계층에 비즈니스 로직을 구현하면 여러 Controller에서 해당 로직을 재사용할 수 있다.
- 테스트 용이성 : Controller는 비교적 단위테스트가 간단한데 DB 조회를 구현하면 테스트가 복잡해진다.

의 문제가 발생할 수 있다.

따라서 데이터베이스 조회와 같은 비즈니스 로직은 주로 서비스 계층에서 수행하고

필요한 경우 서비스 계층에서 캐싱을 구현하여 데이터베이스 조회를 줄이는 것이 좋다.


<br><br>

**코드 설명**

`@Cacheable` 어노테이션의 key 속성은 캐시할 데이터의 key를 지정하고, value 속성은 캐시할 데이터의 value를 지정한다.              

`@Cacheable` 어노테이션에서 key를 설정할 때, SpEL(Sping Expression Language)을 사용하여 동적으로 key를 생성할 수 있다.

이때 SpEL에서는 작은따옴표를 사용하여 문자열을 감싸야 한다.        

<br>

`@Cacheable(key = "#chatMessage.sender", value = "roomId", unless = "#chatMessage.roomId == null")`

`@Cacheable` 어노테이션에서 value를 설정할 때 mehtod의 반환 타입이 void인 경우 

value를 지정할 수 없으므로 value = ""와 같이 빈 문자열로 설정해야 한다.

<br><br>

`redisTemplate.opsForValue().set()`의 마지막 인자로 TTL을 지정하는 방식으로 Redis 캐시에 TTL을 설정했다. 

캐시에서 저장된 데이터의 TTL은 expirationTime 파라미터 값에 따라 설정된다.

<br>

expirationTime을 10으로 설정해서 10시간을 의미하고 redisConfig에서 `.entryTtl(Duration.ofMinutes(30))`로 설정했을 떄

redisTemplate에서 직접 TTL을 지정해주면 해당 TTL로 Redis에 데이터가 저장된다.

따라서, Redis의 데이터는 10시간동안 유지되고, Cache의 데이터는 30분동안 유지된다.

<br><br><br><br>

## Redis 실행

사용할 서비스 method에 annotation 적용

- `@Cacheable` : 적용된 mehtod의 리턴값을 기준으로 캐시에 값을 저장한다.                         
    캐시에 데이터가 없으면 mehtod를 실행하고, 데이터를 추가한다.                         
    이미 데이터가 있으면 캐시의 데이터를 반환한다.                         
    cacheName(value) 값을 기준으로 캐시를 저장하는데, 더 디테일한 값으로 저장하려면 key와 함께 저장한다.                        

- `@CacheEvict` : mehtod가 호출될 때 저장된 캐시를 삭제한다.                                                        
   캐싱된 데이터가 변경될 경우(Insert, Update, Delete) 기존의 캐싱된 데이터가 바뀌어야 하기 때문에 지워주는 작업이 필수이다.                        
                     
- `@CachePut` : 캐시에 값을 저장하는 용도로 @Cacheable과 유사하게 실행 결과를 캐시에 저장하지만,                                                  
   조회 시에 저장된 캐시의 내용을 사용하지는 않고 항상 mehtod의 로직을 실행한다.                        
              
- `@Caching` : 하나의 mehtod를 호출 할 때 Cacheable, CacheEvict 등 여러 개의 캐싱 동작을 수행해야 할 때 사용한다.                        
            
- `@CacheConfig` : 클래스 단위로 캐시 설정을 동일하게 하고 싶을 때 사용한다.                        

<br>

일반 mehtod에 어노테이션만 붙여서 사용하면 별도의 Cache mehtod를 정의할 필요가 없다. 

`@Controller` mehtod에 적용하면 파라미터를 Redis의 키값으로 자동지정된다. 

condition, unless 어노테이션 옵션으로 특정 조건에 따른 캐시적용여부 설정도 가능하다.

<br><br><br>

#### @Cacheable
`@Cacheable` 어노테이션을 사용하지 않으면 캐시 레이어를 거치지 않고 실제 데이터베이스에서 직접 조회하게 된다. 

따라서 조회 속도가 느릴 수 있다. 

`@Cacheable` 어노테이션을 사용하면 먼저 캐시를 조회하고, 캐시에 데이터가 없을 경우에만 실제 데이터베이스에서 조회하므로, 

조회 속도가 빠르고 불필요한 데이터베이스 부하를 줄일 수 있다.

<br><br><br><br>

### 저장과 조회             
String roomId = "123";                   
String nickname = "haedal"; 이라고 할 때,                
(ChatMessage.sender와 nickname은 같은 값)         

```java
@Cacheable(key = "#chatMessage.sender", value = "roomId", unless = "#chatMessage.roomId == null")
public void addRedis(ChatMessage chatMessage){
    long expireTimeInSeconds = 24 * 60 * 60;
    long creationTimeInMillis = System.currentTimeMillis();
    long remainingTimeInSeconds = expireTimeInSeconds - ((System.currentTimeMillis() - creationTimeInMillis) / 1000);
    redisTemplate.opsForValue().set(chatMessage.getSender(), chatMessage.getRoomId(), remainingTimeInSeconds, TimeUnit.SECONDS);
}

@CachePut(value = "roomId", key = "#nickname", unless = "#result == null")
public String getRedis(String nickname){
    return redisTemplate.opsForValue().get("roomId:" + nickname);
}
```
`@Cacheable`로 값을 저장하고 TTL을 설정하여 일정 시간 동안 유효하게 유지한 후, 

조회 시에 `@CachePut`을 사용하여 갱신하는 방식은 캐시의 효율적인 활용을 가능하게 했다. 

<br><br>

#### getRedis null
기존 getRedis에서 `@Caceable`을 사용했으나 값을 제대로 가져올 때가 있고 없을 때가 있었다.

처음엔 [cacheManager](https://haedal-uni.github.io/posts/Redis-Cache/#%EC%BA%90%EC%8B%9C-%EA%B5%AC%EC%84%B1) 설정도 바꿔보고 

이것 저것 코드를 고쳐보다가 `@CachePut`으로 바꾸고 test를 해보니 값을 제대로 가져오기 시작했다.

왜 `@Caceable`에서 `@CachePut`로 수정을 하니 제대로 get이 동작할까?

<br>

`@Cacheable` 어노테이션은 메서드가 호출되기 전에 캐시를 확인하고, 캐시에 저장된 결과가 있으면 해당 결과를 반환한다. 

만약 캐시에 값이 없을 때 null을 반환한다면, 이후에도 항상 null을 반환할 것이다.

`@CachePut` 어노테이션은 메서드의 로직을 실행하고 그 결과를 강제로 캐시에 저장한다. 

매번 메서드가 호출될 때마다 로직이 실행되고 그 결과가 캐시에 갱신된다. 

값이 변경되는 경우에도 항상 메서드의 로직이 실행되고, 그 결과가 캐시에 저장되기 때문에 정상적으로 값을 가져올 수 있다.

따라서 `@Cacheable`을 사용해서 값이 올바르게 가져와지지 않는 문제로 인해 null이 뜬 것이 아닐까? 

<br><br>

terminal에서 keys * 로 조회를 하면 아래와 같이 출력된다. 

1) "roomId::haedal"            
2) "haedal"    

<br><br><br>

#### 결과가 2개?
첫 번째 호출 시 addRedis mehtod가 호출되어 Redis에 "hello" 값을 저장하고 캐시에도 저장된다. 

따라서 roomId::haedal 로그와 "haedal"라는 값이 반환된다.

두 번째 호출 시 getRedis mehtod가 호출되고, 캐시에서 "haedal" 값을 가져온다. 

따라서 "haedal"라는 값이 반환된다.

결과적으로 "roomId::haedal" 로그와 "haedal" 값이 두 번 출력된다.  

<br>

캐싱 기능을 사용할 때 `@Cacheable` 어노테이션이 적용된 mehtod는 캐시에 저장된 값을 반환하고, 

`@CachePut` 어노테이션이 적용된 mehtod는 캐시에 값을 저장하는 역할을 한다.

첫 번째 호출에서 addRedis method가 호출되면서 Redis에 값을 저장하고 캐시에도 저장된다. 

이후에 같은 키(#chatMessage.sender)로 호출되면 `@Cacheable` 어노테이션이 적용된 getRedis mehtod가 호출되어 캐시에서 값을 가져오게 된다.

따라서 첫 번째 호출은 addRedis mehtod가 호출되고 두 번째 호출부터는 getRedis mehtod가 호출되는 것이다.

<br>

만약 getRedis()에 `@Cacheable`을 사용하여 method를 캐시할 경우, 캐시에 값이 존재하면 해당 값을 반환하고 mehtod 실행을 건너뛴다. 

따라서 세 번째 호출에서는 getRedis method가 실행되지 않고, 이전에 캐시에 저장된 값인 "haedal"이 반환될 것이다.
 
<br><br><br><br>

### 저장과 조회가 같이 있는 code

#### Controller
```java
@PostMapping("/room")
@Cacheable(key = "#nickname", value = "createRoom", unless = "#nickname == 'null'", cacheManager = "cacheManager")
public ChatRoomDto createRoom(@RequestBody String nickname) {
    return chatService.createRoom(nickname);
}
```
- `unless = "#nickname == 'null'"` : nickname가 null일 때 캐시를 저장x                                          

- `cacheManager = "cacheManager"` : 위의 config에서 작성한 cacheManager 사용                             

| Element | Description | Type |
| --- | --- | --- |
| cacheName | 캐시 이름 (설정 mehtod 리턴값이 저장되는) | String[] |
| value | cacheName의 alias | String[] |
| key | 동적인 키 값을 사용하는 SpEL 표현식동일한 cache name을 사용하지만 구분될 필요가 있을 경우 사용되는 값 | String |
| condition | SpEL 표현식이 참일 경우에만 캐싱 적용- or, and 등 조건식, 논리연산 가능 | String |
| unless | 캐싱을 막기 위해 사용되는 SpEL 표현식condition과 반대로 참일 경우에만 캐싱이 적용되지 않음 | String |
| cacheManager | 사용 할 CacheManager 지정(EHCacheCacheManager, RedisCacheManager 등) | String |
| sync | 여러 스레드가 동일한 키에 대한 값을 로드하려고 할 경우, 기본 mehtod의 호출을 동기화즉, 캐시 구현체가 Thread safe 하지 않는 경우, 캐시에 동기화를 걸 수 있는 속성 | boolean |

<br><br><br>

#### Service
```java
private final RedisTemplate<String, ChatRoomDto> redisTemplate;
private final Long hours = 10L;

public ChatRoomDto createRoom(String nickname) {
    User user = userRepository.findByNickname(nickname).orElseThrow(UserNotFoundException::new);
    ChatRoomDto chatRoom = new ChatRoomDto();
    if (!chatRepository.existsByUserId(user.getId())) {
        chatRoom = ChatRoomDto.create(nickname);
        ChatRoomMap.getInstance().getChatRooms().put(chatRoom.getRoomId(), chatRoom);
        Chat chat = new Chat(chatRoom.getRoomId(), user);
        chatRepository.save(chat);
        redisTemplate.opsForValue().set(nickname, chatRoom, hours, TimeUnit.HOURS);
        return chatRoom;
    } else {
        ChatRoomDto chatRoomDto = redisTemplate.opsForValue().get(nickname);
        if(chatRoomDto ==null){
            Optional<Chat> findChat = chatRepository.findByUserId(user.getId());
            return ChatRoomDto.of(findChat.get().getRoomId(), nickname, user, lastLine(findChat.get().getRoomId()));
        }
        return ChatRoomDto.of(chatRoomDto.getRoomId(), chatRoomDto.getNickname(), user,lastLine(chatRoomDto.getRoomId()) );
    }
}
```
<br>

ChatRoomDto 클래스를 Serializable 인터페이스를 구현하도록 변경한다.

```java
public class ChatRoomDto implements Serializable { }
```
직렬화(Serialization)란 객체를 바이트(byte) 형태로 변환하는 것을 말한다. 

변환된 바이트 형태의 데이터는 네트워크 상에서 전송되거나 파일로 저장될 수 있다. 

이렇게 바이트 형태로 변환된 데이터를 다시 객체로 역직렬화(Deserialization)할 수 있다.

<br>

Java에서 객체 직렬화는 Serializable 인터페이스를 구현함으로써 가능하다. 

Serializable 인터페이스를 구현하면 해당 클래스는 직렬화가 가능한 클래스가 되며

객체를 바이트 형태로 변환하여 저장하거나 전송할 수 있다.

<br>

객체 직렬화는 네트워크 상에서 객체를 전송하거나, 객체를 파일에 저장하고 불러올 때 사용된다. 

예를 들어, 웹 애플리케이션에서 세션 데이터를 저장하고 불러올 때, 세션에 저장된 객체는 직렬화하여 저장되고, 

다시 불러올 때는 역직렬화하여 객체로 변환된다. 

또한, 캐시에 저장된 데이터를 객체 직렬화하여 저장하면, 캐시를 다시 불러올 때 데이터를 바로 사용할 수 있다.

<br><br><br><br>

### 만료시간 설정

```java
public void addRedis(ChatMessage chatMessage, Long expirationTime){
    long expireTimeInSeconds = 24 * 60 * 60;
    long creationTimeInMillis = System.currentTimeMillis();
    long remainingTimeInSeconds = expireTimeInSeconds - ((System.currentTimeMillis() - creationTimeInMillis) / 1000);
    redisTemplate.opsForValue().set(chatMessage.getSender(), chatMessage.getRoomId(), remainingTimeInSeconds, TimeUnit.SECONDS);
}
```
24시간을 기준으로 현재시간으로 부터 남은 시간을 초 단위로 계산해서 만료시간 설정한다.


<br><br><br>

*reference*            
[Spring Cache, 제대로 사용하기](https://gngsn.tistory.com/157)                   
[[JAVA Spring Boot] Rest API + 레디스 캐시 (Redis Cache) 적용 및 샘플 예제](https://kim-oriental.tistory.com/28)                   
[SpringBoot기반 Redis Cache 활용법](https://yonguri.tistory.com/82)                   
[Cache 기능 Redis로 구현하기](https://pamyferret.tistory.com/25)   

  
