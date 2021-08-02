---
layout: post
title: Slack + Coveralls 연동
catalog: true
tags:
  - Test
date: 2018-03-17
subtitle: Slack
header-img: https://i.imgur.com/avC1Xor.jpg
---

[![Coverage Status](https://coveralls.io/repos/github/dev-cheese/cheese/badge.svg)](https://coveralls.io/github/dev-cheese/cheese)

**본 포스팅은 이미 Travis Ci +  Coveralls 연동이 이루어져있다는 가정하고 Coveralls Slack 연동을 중점으로 다루겠습니다.**
![](https://i.imgur.com/sHagJxo.png)

Coveralls 소스코드의 커러지를 측정 해주는 도구입니다. Coveralls에 대한 자세한 설명은 [Coveralls](https://blog.outsider.ne.kr/954) 를 참조해주세요.

## Slack Web Hook 설정

**Incoming WebHooks 설정을 완료 하신경우 Coveralls 설정 으로 바로 넘어가시면 됩니다. Slack Web Hook 이미 한번 설정을 완료해서 스크린샷 화면이 조금 다를 수 있습니다.**
![](https://i.imgur.com/78QivsI.png)
Slack Apps 페이지에서 **Incoming WebHooks** 를 설치합니다.

![](https://i.imgur.com/i99lWHM.png)
Slack 체널중 Web Hook를 받을 체널을 선택합니다.

![](https://i.imgur.com/0dnlJWv.png)
Webhook URL 값을 확인할 수 있습니다. 이 Webhook URL 값으로 Coveralls 리포팅을 슬랙으로 받아 볼 수 있습니다.


## Coveralls 설정

![1](https://i.imgur.com/YqYWP7U.png)

* Coveralls 연동되있는 Repository 왼쪽 상단의 setting 페이지로 이동합니다.


![2](https://i.imgur.com/Q0vE2HK.png)

1. WEBHOOK URL:  위에서 발급한 `WEBHOOK URL` 값을 입력합니다.
2. CHANNEL : 위에서 생성한 체널를 입력합니다. 필수는 아닙니다.
3. 테스트 버튼을 클릭하여 슬랙으로 위 이미지처럼 메세지가 잘 오는지 확인합니다.
4. SAVE NOTIFICATION 버튼을 누릅니다.

![](https://i.imgur.com/sHagJxo.png)

**설정을 완료하면 Travis CI 에서 배포 작업이 진행이 완료되면 Slack에서 Coveralls 에대한 리포팅 메세지가 자동으로 오게 됩니다.**

