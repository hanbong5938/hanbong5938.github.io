---
layout: post
title: Swagger 날짜 포멧팅
subtitle: Swagger Tip
catalog: true
header-img: 'https://i.imgur.com/avC1Xor.jpg'
tags:
  - Spring
date: 2018-05-03 00:00:00
thumbnail:
---


![](https://i.imgur.com/YGnD90T.png)

Swagger를 API 도큐먼트로 사용하고 계시다면 날짜 관련 model value는 지저분하게 출력 됩니다. 이 문제를 해결 하는 방법에 대해서 간단하게 포스팅 하겠습니다.

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build()
                .directModelSubstitute(Timestamp.class, Long.class);

    }
}
```

directModelSubstitute 메소드는 다음과 같이 작용합니다. 변경하고 싶은 클래스, 변경되고자 하는 포멧팅 입니다.
저같은 경우는 long 타입의 클래스로 날짜를 출력하니 `directModelSubstitute(Timestamp.class, Long.class);`를 적용했습니다.





![](https://i.imgur.com/THkaEN6.png)

작업을 완료하시면 위 그림처럼 표시 됩니다.