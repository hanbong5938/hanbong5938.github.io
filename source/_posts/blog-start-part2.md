---
layout: post
title: 블로그를 시작하려는 당신에게(2)
catalog: true
date: 2017-01-31
header-img: https://i.imgur.com/avC1Xor.jpg
tags:
 - Essay

---


[블로그를 시작하려는 당신에게(1)](https://cheese10yun.github.io/blog-start)</b>에서는 블로그를 선택하는 과정을 말했다면 이번 포스팅은 블로그를 많은 사람이 볼 수 있도록 노출 시키는 방법에 대해서 다루어보겠다.

이왕 시작한 블로그 많은 사람이 보는 게 좀 더 좋지 않겠는가?<del><b>누구나 관종의 피는 가지고 있으니 말이다.</b></del>

**네이버 웹마스터 도구, 구글 웹마스터, 구글 에널틱스를 블로그를 노출 시키고 관리해 보자.**

### 네이버 웹마스터 도구

**네이버 웹마스터**에들어가서 사이트를 추가 시킨다.

![](https://i.imgur.com/iJNq0q1.png)

**등록 절차는 간단하니 다루지 않겠다. (사실 스크린샷을 깜빡하고 못 찍었다.)**


#### 검증에서 웹 페이지 최적화 검사를 진행한다.
![](https://i.imgur.com/IVQHkGT.png)

검색결과가 표시되고 문제가 있는 경우는 빨간색으로 X 표시가 나타난다. ***그 빨간색 표시를 중점으로 개선작업을 진행하면 된다. ?를 클릭하면 친절하게 개선방법을 네이버에서 제안해준다. 대표적인 설정만 짚고 넘어가겠다.***


![](https://i.imgur.com/GfExIWL.png)

##### 오픈 그래프 정보 & 소셜 미디어
오픈 그래프 & 소셜미디어 태그는 사이트가 소셜 미디어로 공유될 때 우선적으로 활용되는 정보로, 아래 그림 처럼 미리 사이트를 보여주는 기능이다.  

![](https://i.imgur.com/e2onpRV.png)
```html
<!--오픈 그래프 정보-->
<meta property="og:type" content="blog">
<meta property="og:title" content="Yun Blog">
<meta property="og:description" content="Node.js.....">
<meta property="og:image" content="https:...images/cover4.jpg" />
<meta property="og:url" content="https://cheese10yun.github.io/">
<!--소셜 미디어-->
<meta name="twitter:card" content="summary" />
<meta name="twitter:title" content="Yun Blog" />
<meta name="twitter:url" content="https://cheese10yun.github.io/" />
<meta name="twitter:image" content="https:../images/cover4.jpg" />
<meta name="twitter:description" content="...." />
```
**해당 테그가 추가안되있는 경우에는 추가시키자.**

![](/img/blog-start-2/2.png)

**블로그에 글이 추가되면 네이버 검색로봇이 웹 페이지에 접근해 정보를 수집해가 자동으로 블로그를 네이버에 노출 시켜준다. 설정은 아주 간단하다.**

##### robots.txt 설정

![](https://i.imgur.com/vyJsG6s.png)

검증 탭의 **robots.txt** 에서 설정이 가능하다.
네이버에서 해당 robots.txt 파일을 작성해주고 그것을 ***복사해서 robots.txt을 root에 위치시킨다.***

작업이 완료되면 robots.txt 검사를 진행한다. 정상 처리되었으면 ~~수집이 가능합니다. 라는 alert 창을 확인할 수가 있을 것이다.

##### RSS 제출
![](https://i.imgur.com/TH3OBbT.png)

요청에 RSS 제출 텝에서 RSS 제출해준다.

#### 웹 페이지 최적화 검사
위의 작업을 완료하면 다시 웹 페이지 최적화 작업을 진행하고 안 돼 있는 부분이 있다면 네이버에서 제안해주는 작업을 진행한다.


### 마무리

생각보다 정리해야할 내용이 많아 구글 웹마스터, 구글 에널틱스는 다음 포스팅에서 다루겠습니다... 분량 조절 실패
