---
layout : post
title : "외부 라이브러리 Bean 등록"
category : Spring
---
```markdown
1. -X Option (모든 VM에서 동작하지 않을 수 있는 비표준 option이며, 버젼별로 언급없이 변경되어질 수 있음)
-Xms : 초기 Heap size 설정
-Xmx : 최대 Heap size 설정
-Xss : 각 Thread에 할당되는 Stack size 설정
-Xmn : New 영역을 위한 Heap size 설정

2. -XX Option (올바른 동작을 위해 특정한 시스템 요구사항들이 있으며, 시스템 설정 파라미터에 대한 접근 권한이 요구됨)
-XX:PermSize : 초기 Permanent size 설정
-XX:MaxPermSize : 최대 Permanent size 설정

```

```bash
java -Xmx32m -Xss256k -jar ./test.jar
```
