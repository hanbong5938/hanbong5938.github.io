---
layout: post
title: npm sudo 권한으로 설치시 오류 해결
date: 2017-02-11
catalog: true
header-img: https://i.imgur.com/avC1Xor.jpg
tags:
- Node
---

```bash
-bash: sodu: command not found
```

NPM으로 모듈을 설치할 때 <code><b>sudo</b></code> 권한으로 설치할 경우 위와 같은 오류가 발생할 경우가 있다.


```bash
sudo ln -s /usr/local/bin/node /usr/bin/node
sudo ln -s /usr/local/lib/node /usr/lib/node
sudo ln -s /usr/local/bin/npm /usr/bin/npm
sudo ln -s /usr/local/bin/node-waf /usr/bin/node-waf
```

위의 명령어를 입력하면 sudo npm으로 설치를 진행할 수 있다.