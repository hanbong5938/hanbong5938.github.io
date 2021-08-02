---
layout: post
title: Jackson 어노테이션 사용법(4)
subtitle: 'Polymorphic Type Handling Annotations,  General Annotations'
catalog: true
header-img: 'https://i.imgur.com/avC1Xor.jpg'
tags:
  - Jackson
date: 2018-06-20 00:00:00
thumbnail:
---


## Polymorphic Type Handling Annotations,  General Annotations
* [Jackson Annotation Examples](http://www.baeldung.com/jackson-annotations) 예제를 적용전, 적용후로 나누어서 정리 해봤습니다.
* 테스트코드도 참고하시면 좋습니다.
* 해당 코드 및 전체 Jackson 정리는 [Github](https://github.com/cheese10yun/blog-sample/tree/master/jackson)를 참고해주세요

## Jackson Polymorphic Type Handling Annotations

* Polymorphic을 통한 직렬화, 비직렬화

```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public static class Zoo {
    public Animal animal;

    public Zoo(Animal animal) {
        this.animal = animal;
    }

    @JsonTypeInfo(
            use = JsonTypeInfo.Id.NAME,
            include = JsonTypeInfo.As.PROPERTY,
            property = "type")
    @JsonSubTypes({
            @JsonSubTypes.Type(value = Dog.class, name = "dog"),
            @JsonSubTypes.Type(value = Cat.class, name = "cat")
    })
    @Getter
    @NoArgsConstructor(access = AccessLevel.PRIVATE)
    public static class Animal {
        private String name;

        public Animal(String name) {
            this.name = name;
        }
    }

    @JsonTypeName("dog")
    @Getter
    @NoArgsConstructor(access = AccessLevel.PRIVATE)
    public static class Dog extends Animal {
        public double barkVolume;

        public Dog(String name) {
            super(name);
        }
    }

    @JsonTypeName("cat")
    @Getter
    @NoArgsConstructor(access = AccessLevel.PRIVATE)
    public static class Cat extends Animal {
        boolean likesCream;
        public int lives;

        public Cat(String name) {
            super(name);
        }
    }

}
```
```json
{
    "animal": {
        "type": "dog",
        "name": "lacy",
        "barkVolume": 0
    }
}

{
    "animal":{
        "name":"lacy",
        "type":"cat"
    }
}
```

## Jackson General Annotations

### @JsonFormat

* 날짜 / 시간 값을 직렬화 할 때 포멧팅을 지정합니다
```java
public static class Event {
    public String name;

    @JsonFormat(
            shape = JsonFormat.Shape.STRING,
            pattern = "dd-MM-yyyy hh:mm:ss")
    public Date eventDate;

    public Event(String name, Date date) {
        this.name = name;
        this.eventDate = date;
    }
}
```
```json
//적용전
{
  "name":"party",
  "eventDate":1419042600000
}
//적용후
{
  "name": "party",
  "eventDate": "20-12-2014 02:30:00"
}
```

### @JsonUnwrapped
* 직렬화, 비 직렬화 될 때 언 래핑 / 병합되어야 하는 값 을 정의하는 데 사용됩니다
```java
public static class UnwrappedUser {
    public int id;

    @JsonUnwrapped
    public Name name;

    public UnwrappedUser(int id, Name name) {
        this.id = id;
        this.name = name;
    }

    public static class Name {
        public String firstName;
        public String lastName;

        public Name(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }
    }
}
```

```json
//적용전
{
  "id": 1,
  "firstName": "John",
  "lastName": "Doe"
}
//적용후
{
  "id":1,
  "name":{
    "firstName":"John",
    "lastName":"Doe"
  }
}
```

### @JsonView
* 속성이 serialization / deserialization에 포함될 View 를 나타내는 데 사용됩니다.
```java
public static class Views {
    public static class Public {
    }

    public static class Internal extends Public {
    }
}

public static class Item {
    @JsonView(Views.Public.class)
    public int id;

    @JsonView(Views.Public.class)
    public String itemName;

    @JsonView(Views.Internal.class)
    public String ownerName;

    public Item(int id, String itemName, String ownerName) {
        this.id = id;
        this.itemName = itemName;
        this.ownerName = ownerName;
    }
}
```
```json
//적용전
{
  "id":2,
  "itemName":"book",
  "ownerName":"John"
}
//적용후
{
  "id":2,
  "itemName":"book"
}
```

### @JsonManagedReference, @JsonBackReference
* 객체의  상위 / 하위 관계를 처리 명시하고 무한 순함참조에러를 해결합니다.
```java
public static class ItemWithRef {
    public int id;
    public String itemName;

    @JsonManagedReference
    public UserWithRef owner;

    public ItemWithRef(int id, String itemName, UserWithRef owner) {
        this.id = id;
        this.itemName = itemName;
        this.owner = owner;
    }
}

public static class UserWithRef {
    public int id;
    public String name;

    @JsonBackReference
    public List<ItemWithRef> itemWithRefs = new ArrayList<>();

    public UserWithRef(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public void addItem(ItemWithRef item) {
        itemWithRefs.add(item);
    }
}
```
```json
//적용전
// Infinite recursion (StackOverflowError)... 무한 순함참조 에러
//적용후
{
  "id":2,
  "itemName":"book",
  "owner":{
    "id":1,
    "name":"John"
  }
}
```

### @JsonFilter
* 직렬화시에 사용되는 필터를 지정합니다.
```java
@JsonFilter("myFilter")
public static class BeanWithFilter {
    public int id;
    public String name;

    public BeanWithFilter(int id, String name) {
        this.id= id;
        this.name = name;
    }
}
```
```json
//적용전
{
  "id":1,
  "name":"My bean"
}
//적용후
{
  "name":"My bean"
}
```