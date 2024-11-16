# Redis Cache 적용 : Redis Cache를 활용한 학습 데이터 TTL 관리

단어 학습 기능을 구현하며 **오늘 학습할 단어**는 **반복 조회**가 가능해야하고

이전(yesterday) 또는 이후 학습할 단어(tomorrow)와 명확히 구분되도록 해야 했다. 

이를 해결하기 위해 **Redis Cache**를 도입했다. 

<br>

캐시를 활용하며 발생할 수 있는 다양한 예외 상황을 고려한 뒤 

자정까지 남은 시간을 TTL(Time-To-Live)로 설정 + 학습한 데이터는 즉시 Study 테이블에 db 저장하는 방식을 채택했다.


<br><br><br>   

## Redis Cache 도입 배경
학습 기능 구현 시 다음과 같은 요구사항을 충족해야 했다.

<br>  

**1.** 오늘 학습할 단어는 당일 동안 계속 조회될 수 있어야 한다.

**2.** 자정 이후에는 새로운 학습 단어 목록이 제공되어야 한다.

**3.** 데이터베이스(DB) 조회를 최소화해야 한다.

<br>

이를 해결하기 위해 Redis Cache를 적용하고 자정까지 남은 시간을 TTL로 설정했다.      

<br><br><br>

```java
[Redis Cache Config]
TTL을 오늘 자정까지 남은 시간으로 설정

[Service]
@Cacheable을 활용하여 db 조회 최소화

<method>
Study 테이블에서 가장 최근 날짜 컬럼을 조회해서
if (만약 오늘 날짜가 아니라면) {
    1. 기존 로직과 동일하게 조회
    2. Study에 해당 목록들을 db에 저장한다.
} else { // 오늘 날짜라면
    Study에서 오늘 날짜에 해당하는 단어 목록을 가져와서 reutrn한다.
}
```

<br><br><br><br>  

## 구현 과정
### 1. Entity
`Study` 테이블에 학습 날짜를 구분하기 위한 `date` 컬럼을 추가했다. 
```java
@Column
private LocalDate date;
```

<br><br>

#### SQL 변경 

```sql
ALTER TABLE study ADD COLUMN date DATE;
```

<br><br><br><br>

### 2. Redis Cache Config 설정
TTL은 오늘 자정까지 남은 시간을 계산하여 설정했다. 
```java
@Configuration
@EnableCaching // 캐시 기능 활성화
public class RedisConfig {

    @Value("${spring.data.redis.host}")
    private String redisHost;

    @Value("${spring.data.redis.port}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(redisHost, redisPort);
    }

    @Bean
    public CacheManager cacheManager() {
        // 자정까지 남은 시간을 계산
        LocalTime midnight = LocalTime.MIDNIGHT;
        Duration ttl = Duration.between(LocalDateTime.now(), LocalDateTime.now().toLocalDate().atTime(midnight).plusDays(1));

        RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig()
                .disableCachingNullValues()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
                .entryTtl(ttl); // 자정까지 남은 시간을 TTL로 설정

        return RedisCacheManager.builder(redisConnectionFactory())
                .cacheDefaults(cacheConfig)
                .build();
    }
}
```

<br><br><br><br>

### 3. DB 조회 쿼리

**1.** 가장 최근 날짜를 조회하기 위해 `max()` 사용

```java
@Query("SELECT max(s.date) FROM Study s")
LocalDate findLastDay();
```

<br><br>

**2.** 오늘 날짜의 학습 데이터 조회 
```java
@Query("select s from Study s where s.date = :today")
List<Study> findLastDayForStudy(LocalDate today);
```

<br><br>

**3.** meaningId에 해당하는 Sentence 데이터 조회 
```java
@Query("select st from Sentence st where st.meaning.id = :meaningId")
Sentence findBySentence(Long meaningId);
```

<br><br><br><br>   

### 4. Service  
```java
@Cacheable(key="#username", value = "getStudyWord", unless = "#result==null", cacheManager = "cacheManager")
public List<StudyResponseDto> getStudyWord(String username) {
    LocalDate date = studyRepository.findLastDay();
    LocalDate today = LocalDate.now();
    if( date==null || !date.isEqual(today) ){ // 오늘 날짜가 아니라면 학습하지 않은 데이터 10개 조회
        // 기존 로직 생략
        studyRepository.saveAll(studyList); // study 테이블에 데이터 저장  
    } else { // 오늘 날짜라면 Study 테이블에서 조회
        List<Study> study = studyRepository.findLastDayForStudy(today);
    }
}
```

`@Cacheable`을 활용해 DB 조회를 최소화했다.    

오늘 날짜와 가장 최근 학습 날짜를 비교해 조건에 따라 DB에서 데이터를 조회하거나 새 데이터를 생성했다.  

<br><br>  

#### TTL 값 검증  
TTL 설정이 제대로 적용되었는지 확인하기 위해 아래 코드를 작성했다. 

```java
LocalTime midnight = LocalTime.MIDNIGHT;
Duration ttl = Duration.between(LocalDateTime.now(), LocalDateTime.now().toLocalDate().atTime(midnight).plusDays(1));
log.info("ttl : " + ttl); // ttl : PT27M40.0414405S (출력 예시)  
```

![image](https://github.com/user-attachments/assets/a97bdb5e-54e9-4dc2-a8e4-4987719db7c8)

- P : "Period"를 나타내는 시작 문자 (기간을 나타냄)    
- T : 시간 정보를 시작하는 표시
- H: 시간(hours)
- M: 분(minutes)
- S: 초(seconds)

27M → 27분

40.0414405S → 40초와 소수점 이하로 0.0414405초를 의미
  
PT27M40.0414405S는 현재 시점에서 자정까지 남은 시간이 27분 40초 정도 남았다고 보면 된다.  

<br><br><br><br>

### Front : LocalStorage TTL 설정
서버에 단어를 요청하기 전에 localStorage에 값이 있는지 체크해 서버 요청을 최소화 시켰었다.

하지만 학습 단어 데이터는 하루 동안 유효하므로 localStorage에 TTL을 설정했다. 

<br><br>

**localStorage TTL 설정**
```js
function getStudyWords(){
    // 일부 코드 생략
    const now = new Date();
    const midnight = new Date(now.getFullYear(), now.getMonth(), now.getDate() + 1, 0, 0, 0); // 다음 날 자정
    const ttl = midnight.getTime() - now.getTime(); // 밀리초 단위 남은 시간
    setTTL(username,JSON.stringify(response), ttl)
}

function setTTL(username, value, ttl){
    const expiry = Date.now() + ttl; // 현재 날짜 + TTL(ms)
    const item = {
        value, // 저장할 데이터
        expiry // 만료 시간
    };
    localStorage.setItem(username, JSON.stringify(item));
}
```

<br><br>


Redis Cache와 LocalStorage를 활용해 학습 단어의 유효 기간을 관리함으로써 서버와 DB의 부하를 최소화했다. 

특히 오늘 자정까지 남은 시간을 TTL로 설정함으로써 학습 데이터의 유효성을 유지하는 동시에 효율성을 확보할 수 있었다.


