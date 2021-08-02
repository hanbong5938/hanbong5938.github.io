---
layout: post
title: Jackson 어노테이션 사용법(2)
subtitle: Jackson Deserialization Annotations
catalog: true
header-img: 'https://i.imgur.com/avC1Xor.jpg'
tags:
  - Jackson
date: 2018-06-01 00:00:00
thumbnail:
---


## Jackson Deserialization Annotations

* [Jackson Annotation Examples](http://www.baeldung.com/jackson-annotations) 예제를 적용전, 적용후로 나누어서 정리 해봤습니다.
* 테스트코드도 참고하시면 좋습니다.
* 해당 코드는 [Github](https://github.com/cheese10yun/blog-sample/tree/master/jackson)를 참고해주세요


### @JsonCreator
* JSON key 와 멤버 필드의 이름이 일치하지 않을 경우 사용합니다.
```json
{
  "id":1,
  "theName":"My bean"
}
```

```java
public static class BeanWithCreator {
    public int id;
    public String name;

    @JsonCreator
    public BeanWithCreator(
            @JsonProperty("id") int id,
            @JsonProperty("theName") String name
    ) {
        this.id = id;
        this.name = name;
    }
}
```

### @JacksonInject
* JSON 데이터가 아닌 값을 주입하는데 사용됩니다.
```json
{
  "name": "My bean"
}
```

```java
public static class BeanWithInject {
    @JacksonInject
    public int id;

    public String name;
}
```

###  @JsonAnySetter
* Map을 이용해서 유연성있게 Deserialization 합니다.
```json
{
  "name": "My bean",
  "attr2": "val2",
  "attr1": "val1"
}
```

```java
public static class ExtendableBean {
    public String name;
    private Map<String, String> properties = new HashMap<>();


    @JsonAnySetter
    public void setProperties(String key, String value) {
        properties.put(key, value);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }


    public Map<String, String> getProperties() {
        return properties;
    }
}
```

### @JsonSetter
* 객체와 맴버필드와 일치하지 않을 경우 유용하게 사용할 수 있습니다.

```json
{
  "id": 1,
  "name": "My bean"å
}
```
```java
public static class MyBean {
    public int id;
    private String name;

    @JsonSetter("name")
    public void setTheName(String name) {
        this.name = name;
    }

    public String getTheName() {
        return this.name;
    }
}
```