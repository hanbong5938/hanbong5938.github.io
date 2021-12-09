Arrays.asList()는 Arrays의 private 정적 클래스인 ArrayList를 리턴한다.

하지만 java.util.Arrays.ArrayList 클래스는 set(), get(), contaions() 의 메서드를 가지고 있지만 원소를 추가하는 메서드는 가지고 있지 않아 사이즈를 바꿀 수 없어
아래의 방법을 사용하여 ArrayList를 새로 선언해야만 추가, 삭제가 가능하다.

```java
List<String> list = new ArrayList(Arrays.asList(array));
```
