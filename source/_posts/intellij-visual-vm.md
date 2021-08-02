---
layout: post
title: 인텔리제이 Visual VM Launcher
catalog: true
header-img: 'https://i.imgur.com/avC1Xor.jpg'
tags:
  - IntelliJ
date: 2020-05-30 00:00:00
subtitle:
---


![](https://github.com/cheese10yun/TIL/blob/master/assets/VisualVM%20Launcher-6.png?raw=true)

`VisualVM`을 사용하면 JVM의  Memory, CPU, Thread의 상태를 쉽게 확인할 수 있습니다. `IntelliJ` 플러그인을 설치하면 보다 쉽게 사용이가능 합니다.

![](https://github.com/cheese10yun/TIL/blob/master/assets/VisualVM%20Launcher-2.png?raw=true)


[https://visualvm.github.io/download.html](https://visualvm.github.io/download.html) 다운로드 받고 압축을 해제 합니다.

![](https://github.com/cheese10yun/TIL/blob/master/assets/VisualVM%20Launcher-3.png?raw=true)

압축 해제이후, `visualvm` 파일은 이후 설정에 필요하기 때문에 별도의 `path`에 보관하시면 됩니다.


![](https://github.com/cheese10yun/TIL/blob/master/assets/VisualVM%20Launcher-1.png?raw=true)

`IntelliJ -> Plugin -> VisualVM Launcher` 검색해서 설치를 진행합니다.

![](https://github.com/cheese10yun/TIL/blob/master/assets/VisualVM%20Launcher-4.png?raw=true)

설치가 완료되면 `Preferences -> Other Settings -> VisualVM Launcher` 설정으로 이동 이후 `VisualVM executable` -> `visualvm` 위에서 다운받은 `path`를 지정합니다. `JDK home` -> `JDK home path`를 지정합니다.

![](https://github.com/cheese10yun/TIL/blob/master/assets/VisualVM%20Launcher-5.png?raw=true)

위설정을 완료한 이후 `Run -> RunWithVisualVM or DebugVisualVM`으로 실행합니다.

![](https://github.com/cheese10yun/TIL/blob/master/assets/VisualVM%20Launcher-6.png?raw=true)

위 설정이 모두 정상적으로 완료되었으면 `VisualVM` 실행이 되는 것을 확인할 수 있습니다.
