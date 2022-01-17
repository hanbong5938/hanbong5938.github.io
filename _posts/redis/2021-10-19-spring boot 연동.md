---
layout : post
title : "spring boot 연동"
category : Redis
---
레디스 라이브러리 추가

```gradle
// https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis
implementation 'org.springframework.boot:spring-boot-starter-data-redis:2.5.5'
```

yml 파일에 설정 추가

```yaml
spring:
  redis:
    host: localhost
		port : 6379 // 디폴트
```

config 파일 생성

```java
@Configuration
@EnableRedisRepositories(enableKeyspaceEvents = RedisKeyValueAdapter.EnableKeyspaceEvents.ON_STARTUP)
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(host, port);
    }

}
```

entity 파일 생성

timeToLive 소멸 시간

@Indexed 검색 가능하도록 해준다
@TypeAlias 설정해주지 않으면 repository 사용시 findAll() 이 (size = 2) 여도 null 리턴

```java
@Entity
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Builder
@RedisHash(value = "refresh_token", timeToLive = ConstantConfig.REFRESH_TOKEN)
@TypeAlias("refresh_token")
public class RefreshToken {

    @Id
    //@Indexed
    private Long id;

    @Enumerated(EnumType.STRING)
    //@Indexed
    private UserType userType;

    //@Indexed
    private String email;

    //@Indexed
    private String value;

    public static RefreshToken of(Employer employer, String token) {
        return RefreshToken.builder()
                .userType(UserType.E)
                .email(employer.getEmail())
                .value(token)
                .build();
    }

    public RefreshToken updateValue(String token) {
        this.value = token;
        return this;
    }

}
```

사용법 2가지

1. **RedisTemplate**

**key:value를 조작할 수 있는 객체를 통해 직접 설정하는 방법**

1. RedisRunner class 생성 (implements ApplicationRunner)
2. @Autowired StringRedisTemplate
3. opsForValue() 메소드 = ValueOperations<K, V> 객체를 반환(Key:value이므로 보통 <String, String>을 사용)
4. 해당 객체의 set(key, value)로 설정
5. 어플리케이션이 실행되면서(ApplictionRunner) 자동으로 key:value를 설정한다.

```
@Component
public class RedisRunner implements ApplicationRunner {

    @Autowired
    StringRedisTemplate redisTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set("name", "kkambi");
        values.set("age", "20");
```

1. **repository 생성**

```java
@Repository
public interface RefreshTokenRepository extends CrudRepository<RefreshToken, Long> {
    RefreshToken findByUserTypeAndEmail(UserType userType, String email);
}
```

save 등 jpa 처럼 사용가능 하지만 안먹히는 쿼리가 좀 있다.
