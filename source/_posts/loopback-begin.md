---
layout: post
title: Loopback 시작하기
catalog: true
date: 2017-05-05
header-img: https://i.imgur.com/avC1Xor.jpg
tags:
- LoopBack
- Node

---
최근 이직을 하게 되어 이렇게 오랜만에 포스팅하게 되었습니다. 바쁘다는 핑계로 미루다 보면 계속 못할 거 같아 많은 준비는 못 했지만 이번 회사에서 처음 접하게 된 Loopback Framework를 간단하게 정리하겠습니다.


**공식 문서에 상세한 내용이 나와 있어 이론보다는 실습 위주로 설명하겠습니다 1개월 정도밖에 사용해보지 않은 프레임워크라 이론적인 측면을 자세히 설명하기가 어려운 점이 있습니다.**

### Loopback 특징

* 정말 빠르게 REST API를 만들수 있습니다.
* 기본적으로 API Document를 Swagger 만들어줍니다.
* CLI를 통해서 편리하게 모델 생성, 정의 접근 권한 등등 작업을 진행할 수 있습니다.
* 다양한 데이터베이스를 ORM으로 쉽게 관리할 수 있습니다.

### Loopback 설치
* 기본적으로 Node, NPM이 설치되있는 환경이라 가정하겠습니다.

```bash
$ npm install -g loopback-cli
```

### Hello world 프로젝트 설정
**루프백의 기초적인 특징을 알아보겠습니다.**
```
$ lb
? 애플리케이션 이름은 무엇입니까? hello-world
? 프로젝트를 포함시킬 디렉토리 이름 입력: hello-world
  create hello-world/
    info 작업 디렉토리를 hello-world(으)로 번경

? 사용하려는 LoopBack 버전은 무엇입니까? 3.x (current)
? 예상하는 애플리케이션 종류는 무엇입니까? hello-world (A project containing a controller, includi
ng a single vanilla Message and a single remote method)
```
* 터미널에서 원하시는 디렉토리 이동 이후 lb 명령어를 입력하고 프로젝트 설정을 이어 갑니다.
* 프러젝트 이름, 디렉토리 이름 등 간단하게 설정합니다.
* 버전은 3.x 최신 버전을 선택합니다.
* 예상하는 애플리케이션 종류는  hello-world를 선택합니다.

### Hello world 프로젝트 실행
```bash
$ cd hello-world/
$ node .
```
* 위에서 `CLI`로 생성한 프로젝트 디렉토리로 이동합니다.
* `node .` 명령어로 프로젝트를 실행 합니다.
* `http://0.0.0.0:3000/explorer`로 이동합니다.
* API Document `Swagger`가 정상적으로 출력되면 아래 그림과 같습니다.

### API Document
![](http://i.imgur.com/D9cINil.png)

* Loopback 기본 `User`모델을 기반으로 User API를 만들어 줍니다.
* User API는 회원 가입, 로그인, 로그아웃, 회원 정보 수정, 등등 User에 대한 REST API가 있습니다.

#### POST /Users (회원 가입)

![](http://i.imgur.com/ako0hjq.png)
```json
{
  "realm": "string",
  "username": "loopback",
  "email": "loopback@loopback.com",
  "password": "loopback"
}
```

* `data`에 JSON 타입으로 해당 데이터를 입력합니다.
* ***기본설정인 In-memory db에 저장됩니다.***

#### POST /Users/login (로그인)

![](http://i.imgur.com/0W8k2M9.png)

```json
{
  "email": "loopback@loopback.com",
  "password": "loopback"
}
```


* `credentials` 위에서 가입한 email, password 정보를 JSON 타입으로 입력합니다.
* 회원 정보가 일치할 경우 `Response Body`에 `AccessToekn` 정보를 넘겨 줍니다.
* `AccessToekn.id`의 값 `KIjxd....`을 오른쪽 상단 ToKen Set에 입력합니다.
* loopback에서는 기본적으로 인증처리를 AccessToekn 방식으로 지원합니다.
* `AccessToekn` 모델 또한 Loopback의 기본 제공 모델중 하나입니다.

#### GET /Users/{id} (해당 회원 조회)

![](http://i.imgur.com/gWBT25M.png)

* `id` 파라마터에 `AccessToekn`에서 발급 받은 `userId`를 입력합니다.
* 자신의 정보를 조회할 수 있습니다.
* 자신의 이외의 회원 정보를 조회할 경우 아래와 같은 `StatusCode` 401를 리턴 받습니다.

```json
{
  "error": {
    "statusCode": 401,
    "name": "Error",
    "message": "권한 필수"
  }
}
```

### 결론
루프백에서 기본적으로 모델을 생성하고 모델 간의 관계를 정의하면 기본적인 CURD REST API를 자동으로 만들어 줍니다. 또한, Swagger를 이용해서 API Document 또한 자동으로 만들어 주어 정말 빠르게 API를 개발할 수 있게 해줍니다. 또 특정 API에 대한 접근 권한 및 인증 처리도 정말 간단하게 이루어지고, 이 밖에도 다양한 장점들로 빠르게 계발할 수 있도록 도와줍니다. 이러한 장점들을 한 번에 소개하기는 힘들어 해당 파트 마다 소개를 이어 나갈 거 같습니다. 오늘 포스팅한 내용은 부실하지만 이렇게라도 시작을 하지 않으면 계속 늦어질 거 같아 빠르게 정리해보았습니다. 앞으로는 간단한 게시판을 만들면서 Loopback의 장점들을 소개할 예정입니다.
