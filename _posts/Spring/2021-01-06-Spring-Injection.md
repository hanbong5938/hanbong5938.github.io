---
layout : post
title : "Injection 방법"
category : Spring
---

필드주입 사용

```java
@Autowired
private DependencyA dependencyA;

@Autowired
private DependencyB dependencyB;

@Autowired
private DependencyC dependencyC;
```

생성자 주입

```java
private final DependencyA dependencyA;
private final DependencyB dependencyB;
private final DependencyC dependencyC;

@Autowired

public DI(DependencyA dependencyA, DependencyB dependencyB, DependencyC dependencyC) {
	this.dependencyA = dependencyA;
	this.dependencyB = dependencyB;
	this.dependencyC = dependencyC;
	
}
```

setter 주입

```java
private DependencyA dependencyA;
private DependencyB dependencyB;
private DependencyC dependencyC;

@Autowired
public setDI(DependencyA dependencyA, DependencyB dependencyB, DependencyC dependencyC) {
	this.dependencyA = dependencyA;
	this.dependencyB = dependencyB;
	this.dependencyC = dependencyC;
};
```

어노테이션 사용

```java
@RequiredArgsConstructor
public class DI {
	private final DependencyA dependencyA;
	private final DependencyB dependencyB;
	private final DependencyC dependencyC;
}
```

스태틱인 경우 setter 주입

```java
public static class DI {
	private static DependencyA dependencyA;
	private static DependencyB dependencyB;
	private static DependencyC dependencyC;
	
	@Autowired
	public void setDependency(
		DependencyA dependencyA,
		DependencyB dependencyB,
		DependencyC dependencyC
	) {
		DI.dependencyA = dependencyA;
		DI.dependencyB = dependencyB;
		DI.dependencyC = dependencyC;
	}
}
```

아래의 경우 Spring에서 추천하는 생성자 주입을 이용한 방식이다.

> 스프링 4.3의 경우, 단일 생성자를 이용하는 경우에는 @Autowired를 사용하지 않아도 된다.

### 순환 의존성 확인

필드 주입으로 순환 의존성을 파악하기는 어렵습니다. 생성자 주입을 하게 되면 서버 기동시 순환 의존성을 가지는 요소들을 파악할 수 있게 에러 메세지를 표시 하면서 서버 기동이 되지 않는다.

### 불변성

생성자 주입에서 final을 붙이는 이유이다. 필드 주입은 final를 선언할 수 없지만 생성자 주입은 final를 선언함으로써 객체가 변하지 않도록 방지해준다.

### 단일 책임 원칙 위반 확인

Lombok을 사용하지 않았을때 필드 주입을 하게 되면 코드량이 상당히 많아지고 생성자를 확인하여 얼마나 많은 요소들을 사용하지는 한눈에 파악이 가능해진다. "클래스는 한 개의 책임을 가진다" 단일 책임 원칙 위반을 확인할 수 있다.
