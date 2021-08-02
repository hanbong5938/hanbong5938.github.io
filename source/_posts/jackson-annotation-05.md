---
layout: post
title: Jackson 어노테이션 사용법(5)
subtitle: 'Custom Jackson Annotation, Disable Jackson Annotation'
catalog: true
header-img: 'https://i.imgur.com/avC1Xor.jpg'
tags:
  - Jackson
date: 2018-06-23 00:00:00
thumbnail:
---



## Custom Jackson Annotation, Disable Jackson Annotation
* [Jackson Annotation Examples](http://www.baeldung.com/jackson-annotations) 예제를 적용전, 적용후로 나누어서 정리 해봤습니다.
* 테스트코드도 참고하시면 좋습니다.
* 해당 코드 및 전체 Jackson 정리는 [Github](https://github.com/cheese10yun/blog-sample/tree/master/jackson)를 참고해주세요

## Custom Jackson Annotation

* Annotation 직접 정리 할 수 있습니다.

```java
@CustomAnnotation
public static class BeanWithCustomAnnotation {
    public int id;
    public String name;
    public Date dateCreated;

    public BeanWithCustomAnnotation(int id, String name, Date dateCreated) {
        this.id = id;
        this.name = name;
        this.dateCreated = dateCreated;
    }
}

@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonPropertyOrder({"name", "id", "dateCreated"})
@interface CustomAnnotation {
}
```
```json
// 적용전
{
  "id": 1,
  "name": "My bean",
  "dateCreated": null
}
// 적용후, property order 변경, null 값 비 직렬화
{
  "name": "My bean",
  "id": 1
}
```

## Disable Jackson Annotation

* 모든 Jackson annotation 비활성화 하는 방법

 ```java
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonPropertyOrder({"name", "id"})
 public static class MyBean {
     public int id;
     public String name;

     public MyBean(int id, String name) {
         this.id = id;
         this.name = name;
     }
 }
mapper.disable(MapperFeature.USE_ANNOTATIONS); // 모든 Jackson annotation 비활성화
 ```
 ```json
 // MapperFeature.USE_ANNOTATIONS 적용전
 {
     "id":1,
     "name":null
 }

 // MapperFeature.USE_ANNOTATIONS 적용후
 {
   "id": 1
 }
 ```