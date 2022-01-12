---
layout : post
title : "Message.properties 특수문자"
category : Spring
---
```properties
message.sample={{name}}
```

위와 같이 중괄호를 사용하는 경우 {name} 관련 에러가 발생한다.

{0}과 같이 파라미터로 인식하여 발생하는 문제이다.

```bash
message.sample='{{name}}'
```

으로 사용하면 정상적으로 인식된다.

만약 작은 따움표를 사용하고 싶은경우

```properties
sample=''안녕''
```

'' 로 사용하면된다
