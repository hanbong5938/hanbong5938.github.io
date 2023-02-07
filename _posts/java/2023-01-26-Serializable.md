---
layout : post
title : "Serializable"
category : Java
---

## 의미

- 자바 시스템 내부에서 사용하는 Object 또는 Data를 외부의 자바 시스템에서도 사용할 수 있도록 바이트 형태로 데이터를 변환하는 기술(JVM의 메모리에 상주되어 있는 객체 데이터를 바이트 형태로 변환)
- 다차원의 자료를 파일로 저장하거나 네트워크로 보내기에 알맞게 일차원으로 변환하고 다시 원래대로 되돌리는 것을 직렬화라고 한다.
- 역직렬화는 직렬화의 반대를 의미한다.(바이트 => data, object)

## 종류

### `schematic`

- 데이터 모델이 고정되어 직렬화 포멧을 만들어두고 데이터 모델마다 생성해서 쓴다.
- 성능이나 크기를 기준으로 선택하는데 두가지 모두 `schemaless` 방법보다는 효율적이다.

### `schemaless`

- 범용적인 데이터 모델에서 사용하는 접근 방법이다.
- 재귀가 없다는 특징을 가지고 있으며 순서 및 순열이 없는 집합만 가진다. (ex. JSON)
-

## 특징

- Serializable를 구현하게 되면 직렬화 대상인 것을 JVM에게 알려주는 역할만 하기 때문에 구현해야하는 메서드는 존재 하지 않는다.
- 직렬화 작업이 아닌 직렬화 대상인 것을 알려주는 것이기에 성능상에 문제를 일으킬 가능성이 낮다.
- 자바 직렬화를 이용해서 외부 데이터를 저장하면 자바에서만 사용가능하다.

## `serialVersionUID`

- 직렬화 과정에서 serialVersionUID가 포함 되고, 역직렬화 과정에서 Java class에 선언되어 있는 serialVersionUID의 버전과 서로 동일한지를 체크를 한다.
- 필수 값은 아니며 선언하지 않는 경우 클래스의 기본 해쉬 값을 사용한다.
- 컴파일러 구현에 따라 기본 해쉬 값이 달라지기 때문에 필수 값이 아니여도 직접 선언해 주는 것이 좋다.
- 잠재적으로 네트워크를 통해 전송되거나 파일로 저장할 가능성이 조금이라도 있는 클래스에 적용하는 것이 좋다.
- 멤버 변수명은 같은데 멤버 변수 타입이 바뀔 때, serialVersionUID를 변경하지 않으면 역직렬화 과정에서 에러가 발생한다.
- 원시 타입으로 변경하는 경우, properties 혹은 메서드가 삭제, 추가 되는 경우 serialVersionUID를 변경하지 않으면 역직렬화 과정에서 에러가 발생한다.

## `enum`

- enum의 모든 요소를 직렬화할 수 있는 경우 자동으로 직렬화한다.
- enum에 직렬화할 수 없는 필드가 포함된 경우, 직렬화할 수 없다.
- Serializable 인터페이스를 구현하여 명적으로 선언하는 경우 기본 구현 변경되더라도 일관되게 직렬화하고 역직렬화할 수 있도록 보장하는 데 도움이 된다.

## `DTO`

- 네트워크나 디스크를 통해 직렬화가 필요한 형식으로 보내거나 저장하는 경우에는 직렬화하는 것이 좋다. 그렇지 않은 경우 직렬화할 필요가 없다.

## `abstract`

- 직렬화가 필요하지 않은게 일반적이다.
- abstract 클래스가 직렬화 가능한 클래스의 부모 클래스인 경우 부모 클래스의 상태도 자식 클래스의 상태의 일부이므로 자식 클래스가 직렬화된 경우에는 직렬화되어야 한다

## Example

```java
public class Demo implements Serializable {
    private static final long serialVersionUID = 1L;

    private String name;

    // getter, setter
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    protected Demo() {
        // 인자가 없는 public constructor
    }
}
```

