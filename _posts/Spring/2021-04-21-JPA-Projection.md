---
layout : post
title : "JPA Projection"
category : Spring
---
주로 통계와 같이 쿼리를 이용하여 값을 추출하는데

Entity는 없는 값을 조회해서 반환해야할 경우 사용 한다.

```sql
public interface SearchResult {
    String getName();

    String getPhone();
}
```

get형식으로 이름지정해야만 사용할 수 있다.

쿼리에서는 as name 으로 주어져야한다.
