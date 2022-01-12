---
layout : post
title : "SingleTon"
category : Java
---
**정의**

클래스의 오브젝트 개수를 제한시키는 방법론으로, 하나의 클래스당 한 개의 오브젝트만 존재

- 불필요한 메모리 누수를 방지
- 공통된 오브젝트를 사용해야 하는 상황에서 하나의 오브젝트만 사용 (예: DBConnectionPool)

**자바 싱클톤**

공통적 특징

1) 생성자를 private으로 선언하여 외부에서 클래스의 오브젝트를 생성하는 것이 불가능하다

2) 메서드는 static으로 정의하여 모든 영역에서든 접근이 가능하도록 된다

**구현 방법**

if 문을 사용하는 이유는 계속해서 synchronized 하여 속도 저하를 방지하기 위하여

```java
public static JavaSingleton getInstance() 
  { 
    if (instance == null)  
    { 
      synchronized (JavaSingleton.class) 
      { 
        if(instance==null) 
        { 
          instance = new JavaSingleton(); 
        } 
        
      } 
    } 
    return instance; 
  }
```

**Bill Pugh 구현법** : 클래스 내부에 static class 선언

가장 보편적으로 사용

static 키워드로 선언된 객체는 메모리 할당이 단 한번만 이루어지고 final 키워드로 선언되면 그 값을 덮어쓸 수 없음을 이용

getInstance()가 최초로 BillPughSingleton클래스의 instance를 호출한 순간 해당 클래스는 최초 인스턴스를 생성하고, 그 후부터는 호출될때마다 해당 메모리번지에 저장된 인스턴스를 리턴

```java
public class JavaSingleton
{

  private JavaSingleton()

  private static class BillPughSingleton
  {
    private static final JavaSingleton instance = new JavaSingleton();
  }

  public static JavaSingleton getInstance()
  {
    return BillPughSingleton.instance;
  }
}
```

### 스프링 싱클톤

**스프링 컨테이너에 의해서 구현**

@Bean이 정의되면, 스프링 컨테이너는 그 클래스에 대해 딱 한 개의 인스턴스를 만든다. 이 공유 인스턴스는 설정 정보에 의해 관리되고, bean이 호출될 때마다 스프링은 생성된 공유 인스턴스를 리턴 시킨다.

bean 관리의 주체인 스프링 컨테이너는 그 어떤 호출에도 단일 공유 인스턴스를 리턴시키므로 thread safety도 자동으로 보장된다.

1. 자바 싱글톤 클래스로더에 의해 구현

   스프링의 싱글 톤은 스프링 컨테이너에 의해 구현

2. 자바 싱글톤의 scope는 코드 전체

   스프링 싱글톤의 scope는 해당컨테이너 내부

3. 스프링에 의해 구현되는 싱글톤패턴은 Thread safety를 자동으로 보장한다.
   자바로 구현하는 싱글톤패턴은 개발자의 로직에 따라 thread safety를 보장할수도, 보장하지 않을수도 있다.
