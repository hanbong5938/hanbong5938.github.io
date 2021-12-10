---
layout : post
title : "Timezone 안맞는 문제"
category : Spring
---
```markdown
Os - JVM - JPA - DB
```

위처럼 다양한 곳에서 timezone을 가지고 있어 엉키는 문제가 생겨 따로 설정이 필요하다.

JPA의 시간이 설정
```properties
spring.jpa.properties.hibernate.jdbc.time_zone=UTC
```

JVM의 시간이 설정
```java
TimeZone.setDefault(TimeZone.getTimeZone("UTC"));
```

JVM이 설정
```bash
java -jar -Duser.language= -Duser.country=KR -Duser.timezone=Asia/Seoul
```

JPA의 경우 DB에 입력되는것 조정하고

JVM은 어플리케이션에 사용되는 시간이다.
