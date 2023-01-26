---
layout : post
title : "Introspection"
category : Java
---

## 의미 
- JavaBean의 구조를 파악해 properties, methods 와 events를 파악
- Reflection 기법을 사용하며 runtime에 진행된다.
- introspect의 대상인 클래스를 JavaBean의 디자인 패턴에 맞게 작성해야 한다.
- JavaBean의 properties, methods, events에 대해 추가적인 정보를 제공하고 싶으면 BeanInfo 인터페이스의 구현체를 제공하면 된다.


## `Introspector`
- JavaBean의 properties, methods, events를 분석하는 클래스이다.
- BeanInfo가 제공되지 않는 경우, Introspector은 Reflection을 통해 JavaBean의 구조를 분석한다.
- JavaBean의 BeanInfo 클래스들을 캐싱함으로써 성능을 높일 수 있다.


