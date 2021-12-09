---
layout : post
title : "jvm 원격 디버깅"
category : Spring
---

![image](https://user-images.githubusercontent.com/51283645/145336499-32dc9f2d-2758-491f-a634-83771652074a.png)

인텔리제이를 기준으로

remote jvm debug 에서

use module 에서 디버깅하려는 모듈을 선택해준 다음

원격 서버에서 아래와 같이 설정해준다.

```bash
java agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -jar -Dspring.profiles.active=dev ./temp-0.0.1.jar
```

와 같이 jar 파일 실행시

```bash
agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```

을 추가 해준다.
