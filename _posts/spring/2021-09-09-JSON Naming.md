---
layout : post
title : "JSON naming"
category : Spring
---

Spring에서 기본 설정으으로 모델을 만들고 Reponse를 확인해보면 `camelCase`가 적용되어 있는 것을 볼수 있다.

### application.properties
SpringBoot에서 `application.properties`에 아래의 설정을 추가 해 준다

```
spring.jackson.property-naming-strategy=LOWER_CAMEL_CASE
```
`LOWER_CAMEL_CASE`, `LOWER_CASE`, `KEBAB_CASE` 등을 설정해주면 전체적으로 적용이 된다.

### JasonBuilder
```java
public class Jackson2ObjectMapperBuilder {
    @Bean 
    public Jackson2ObjectMapperBuilder jacksonBuilder() {
        Jackson2ObjectMapperBuilder b = new Jackson2ObjectMapperBuilder();
        b.propertyNamingStrategy(PropertyNamingStrategy.LOWER_CAMEL_CASE);
        return b;
    }
}

```

### JsonProperty
```java
@Data
public class Student {

    @JsonProperty("my_name")
    private String myName;

    @JsonProperty("my_age")
    private String myAge;

    @JsonProperty("my_country")
    private String myCountry;

}
```


### JsonNaming (Deprecated)
```
@JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class)
@Data
public class Student {
    private String myName;
    private String myAge;
    private String myCountry;
}
```

`@JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class)`를 붙여주면 변경할 수 있으나

현재 위의 방법은

[https://github.com/FasterXML/jackson-databind/issues/2715](https://github.com/FasterXML/jackson-databind/issues/2715)

멀티 스레드 환경에서 문제로 Deprecated 되었다.

