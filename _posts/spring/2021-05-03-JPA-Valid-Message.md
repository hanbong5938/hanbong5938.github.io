---
layout : post
title : "JPA Valid Mesaage"
category : Spring
---
ValidationMessages.properties 를 생성한다.

```properties
sample=샘플 메세지 입니다.
```

언어별로 설정 원한다면 아래와 같이 파일을 만든다.
```
ValidationMessages_en.properties

ValidationMessages_ko.properties
```


```java
public class Dto {
    @Size(min = 2, max = 22, message = "{sample}")
    private String name;
}
```

