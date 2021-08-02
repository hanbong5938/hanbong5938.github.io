---
layout: post
title: Jackson 어노테이션 사용법(3)
subtitle: Jackson Property Inclusion Annotations
catalog: true
header-img: 'https://i.imgur.com/avC1Xor.jpg'
tags:
  - Jackson
date: 2018-06-13 00:00:00
thumbnail:
---


## Jackson Property Inclusion Annotations

* [Jackson Annotation Examples](http://www.baeldung.com/jackson-annotations) 예제를 적용전, 적용후로 나누어서 정리 해봤습니다.
* 테스트코드도 참고하시면 좋습니다.
* 해당 코드 및 전체 Jackson 정리는 [Github](https://github.com/cheese10yun/blog-sample/tree/master/jackson)를 참고해주세요

### @JsonIgnoreProperties

* 무시할 속성이나 속성 목록을 표시하는 데 사용됩니다
```java
@JsonIgnoreProperties({"id"})
public static class BeanWithIgnore {
    public int id;
    public String name;
}
```

```json
{
  "name": "yun"
}
```

### @JsonIgnore

* 필드 레벨에서 무시 될 수있는 속성을 표시하는 데 사용됩니다.
```java
public static class BeanWithIgnore {
    @JsonIgnore
    public int id;
    public String name;
}
```

```json
{
  "name": "yun"
}
```

### @JsonIgnoreType
* 주석이 달린 형식의 모든 속성을 무시하도록 지정하는 데 사용됩니다 
```java
public static class User {
public int id;
public Name name;

    @JsonIgnoreType
    public static class Name {
        public String firstName;
        public String lastName;
    }
}
```
```json
{
  "id": 1
}
```

### @JsonInclude
* 어노테이션 속성을 제외 하는 데 사용 됩니다 

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
@AllArgsConstructor
public static class MyBean {
    public int id;
    public String name;
}
```
```json
//NON_NULL 사용시 name이 null인 경우에 제외 됩니다.
{
  "id": 1
}
```

### @JsonAutoDetect

```java
@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.ANY)
public static class PrivateBean {
    private int id;
    private String name;
}
```

```json
// Visibility.ANY 경우 표시
{
  "id": 1,
  "name": "yun"
}
```
