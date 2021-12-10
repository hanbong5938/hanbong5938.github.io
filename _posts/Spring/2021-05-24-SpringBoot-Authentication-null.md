---
layout : post
title : "SpringBoot2.4 Migration시 authentication null"
category : Spring
---
Spring boot 2.4.0에서

`@AuthenticationPrincipal` 을 사용하는 경우 authentication의 값이 null이 나온다.

```java
public class Test {
    @GetMapping("/test")
    public TestDto test(@AuthenticationPrincipal Authentication authentication) {
        System.out.println(authentication);
    }
}
```
`@AuthenticationPrincipal` annotation을 제거하면 정상적으로 작동한다.
