---
layout: post
title: 실무에서 Lombok 사용법 - lombok.config
subtitle: 안전하게 Lombok을 사용해보자
catalog: true
header-img: 'https://i.imgur.com/avC1Xor.jpg'
tags:
  - Lombok
  - Guide
date: 2018-12-22 16:00:00
thumbnail:
---



**해당 코드는 [Github](https://github.com/cheese10yun/blog-sample/tree/master/lombok)에 공개되어 있습니다.**

## lombok.config 설정

lombok.config 설정 파일을 통해서 lombok 어노테이션을 제한 할 수 있습니다. [이전 포스팅에](https://cheese10yun.github.io/lombok/)서 언급한 `@Data` 등 사용을 했을 경우 위험 부담이 있는 어노테이션들은 해당 설정에서 제한 할 수 있습니다.


```config
lombok.Setter.flagUsage = error
lombok.AllArgsConstructor.flagUsage = error
lombok.ToString.flagUsage = warning
lombok.data.flagUsage= error
```
**lombok.config 파일을 작성한뒤 Proejct root path에 위치시킵니다**

`lombok.{해당어노테이션}.flagUsage = [warning or error]` 이러한 규칙으로 lombok 어노테이션들을 설정 할 수 있습니다.

![](https://github.com/cheese10yun/blog-sample/blob/master/lombok/assets/lombok-config.png?raw=true)

실제 컴파일을 진행하면 위에서 설정한 `lombok.config`에서 제한한 어노테이션은 warning, error로 표시됩니다.
이 처럼 명확한 가이드 라인이 있으면 설정 파일을 통해서 제한하는 것이 바람직하다고 생각합니다.