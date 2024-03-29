---
layout : post
title : "인증 정보 저장에 로컬 스토리지 보다 쿠키를 선호하는 이유"
category : Browser
---

# 로컬 스토리지
### 장점
- 순수 자바 스크립트로 백엔드 없이 사용이 가능하다.
- 5mb 까지 저장이 가능하다.
### 단점
- XSS 공격에 취약하다. 쿠키와 달리 브라우저에서 js로 직접 접근이 가능하기 때문이다.

# 쿠키
### 장점
- httpOnly 와 보안된 쿠키를 사용한다면 js 를 이용하여 접근이 불가능하다. (XSS 로부터 안전하다는 의미는 아니다.)
- 서버에서 쿠키를 설정할 수 있다.
- 만료 날짜를 설정 할 수 있다.

### 단점
- 4kb 까지 저장이 가능하다.


## 결론
- 쿠키와 로컬 스토리지 모두 XSS 공격에 취약하다.
- 쿠키는 서버에서 설정이 가능하고 만료 날짜를 설정 할 수 있기 때문에 로컬 스토리지 보다 쿠키를 선호한다.
- 사이즈가 큰 경우 등이 있어 두 가지를 비교해 보고 사용하는 것이 유용하긴 하나 일반적으론 쿠키가 선호된다.
