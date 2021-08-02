---
layout: post
title: Package.json 파일로 API 버전 설정
date: 2017-10-14 01:31:24
subtitle:
catalog: true
header-img: https://i.imgur.com/avC1Xor.jpg
thumbnail: https://i.imgur.com/cNL3e5B.png
tags:
- Node
---

## 프로젝트 구성
`package.json` 파일로 API URL을 설정하는 간단한 방법을 포스팅 해보겠습니다. 

![](https://i.imgur.com/cNL3e5B.png)

기본 프로젝트 구성은 WebStorm 으로 Node 프로젝트를 생성한 구조 입니다.

## pacakge.json 파일

```json
{
  "name": "api-version",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {}
}
```
`npm` 의존성 모듈을 관리하는 `pacakge.json` 내용입니다. 해당 내용을 보시면 `version` 이라는 항목이 있습니다. 해당 프로젝트의 버전을 표시하는 정보이며 초기 설정은 `0.0.0` 입니다. 이 `version` 을 활용하여 URI path의 API 버전 설정을 진행하겠습니다.

## app.js
프로젝트 구조에 따라 다소 차이가 있겠지만 각종 라운팅 설정 및 미들웨어 설정하는 `.js` 파일에서 설정을 진행하겠습니다.
```javascript
const packageFile = require('./package.json');
const API_VERSION = packageFile.version.split('.').shift();
const BASE_URL = `/api/v${API_VERSION}`;
```

* packageFile을 require 합니다.
* API_VERSION 변수에 `package.json` 파일의 version 정보 최상단 앞자리만 가져옵니다.
* BASE_URL 변수에 API 버전 정보를 할당합니다.

## middleware 적용

### 기존 middleware
```javascript
app.use('/', index);
app.use('/users', users);
```

### 변경 middleware
```javascript
app.use(`${BASE_URL}/`, index);
app.use(`${BASE_URL}/users`, users);
```
* 위에서 만든 변수 `BASE_URL`를 활용해서 미들웨어 URL 을 변경합니다.

![](https://i.imgur.com/4j8x1TW.png)

* 정상적으로 작동합니다.

## API 버전 변경

```json
{
  "name": "api-version",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {}
}
```
* package.json 의 version 을 변경하면 BASE_URL 가 변경되어 URL 이 변경됩니다.


## 마무리
사실 평소에는 `package.json` version 정보를 전혀 사용하지 않다가 LoopBack 프레임워크에서 위와 같은 방법으로 API 버전 관리를 하는 것을 보고 포스팅을 해보았습니다. 다른 분들은 어떻게 API 버전 관리를 하는지는 잘 모르겠지만 꽤 직관적이며 쉽게 적용 가능한 부분이라고 생각이 듭니다.
