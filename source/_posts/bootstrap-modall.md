---
layout: post
title: Bootstrap Modal 입력값 초기화
catalog: true
date: 2017-01-10
header-img: https://i.imgur.com/avC1Xor.jpg
---

![EC2 Innound](https://i.imgur.com/SHu3JYX.png)

부트스트랩 모달에서 값을 입력하고 모달을 닫으면 이전 입력값은 초기화 되지 않고 남아 있습니다. 이 입력값을 모달을 닫을때 같이 초기화 시켜주는 예제 및 활용법 에대해서 포스팅을 진행하겠습니다.

```javascript
$('.modal').on('hidden.bs.modal', function (e) {
    console.log('modal close');
  $(this).find('form')[0].reset()
});
```


### 활용방법

해당 로직은 공통으로 사용될 로직이니 공통적으로 사용되는 로직체 추가하는 것이 좋습니다.

  생각 보다 내용이 없네요.....
