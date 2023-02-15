---
layout : post
title : "response null 인 경우 생략"
category : Spring
---

# 1. DTO 마다 직접 적용 하는 경우

원하는 property 에 @JsonInclude(JsonInclude.Include.NON_NULL) 를 적용한다.
만약 @JsonInclude(JsonInclude.Include.NON_EMPTY) 를 적용하면 빈 문자열도 생략한다.

```java

@Getter
@Setter
public class Demo implements Serializable {

    private Long id;

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private String name;

}
```

# 2. 전역 설정 하는 경우
직접 Bean 으로 등록하여 사용한다.
JsonInclude.Include.NON_NULL 대신 JsonInclude.Include.NON_EMPTY 를 적용하면 빈 문자열도 생략한다.

```java

@Configuration
public class MapperConfig {

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL); 
        return mapper;
    }
}
```


# 3. yaml 설정을 사용하는 경우
yaml 설정을 사용하는 경우에는 다음과 같이 설정한다.
```yaml
spring:
  jackson:
    default-property-inclusion: non_null
```
