---
layout : post
title : "EntityGraph 사용하여 LazyLoading 데이터 가져오기"
category : Spring
---
```java
@EntityGraph(attributePaths = {"a", "b"}, type = EntityGraph.EntityGraphType.LOAD)
Page<Temp> findById(Pageable pageable);
```

위와 같이 attributePaths 에 원하는 데이터를 설정하면 getA()로 데이터를 불러 올 수 있다.

a와 b는 아래와 같이 도메인에서 사용하는 이름이다

```java
class Temp {
    private A a;
    private B b;
}
```
