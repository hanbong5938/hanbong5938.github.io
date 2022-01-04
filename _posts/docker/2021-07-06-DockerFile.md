---
layout : post
title : "DockerFile"
category : docker
---

### 기본적 구조
```dockerfile
Dockerfile 설명

FROM openjdk:8 이미지명

ADD [파일위치]  [파일별명]

ENV 환경변수

ENTRYPOINT 파라미터

```

### 도커 파일 실행
app → 이름 설정

./ 도커 파일 위치

이미지 생성

```bash
docker build --tag app ./
```



### jdk
```dockerfile
FROM  openjdk:8
ADD ./api.jar api.jar
ENV JAVA_OPTS=""
ENTRYPOINT ["java", "-jar","-Duser.country=ID", "-Duser.timezone=Asia/Jakarta" , "-Dspring.profiles.active=local_in", "/api.jar"]
```

### node

```dockerfile
#어떤 이미지로부터 새로운 이미지를 생성할지를 지정
FROM node:6.2.2

#Dockerfile 을 생성/관리하는 사람
MAINTAINER Jaeha Ahn <eu81273@gmail.com>

# /app 디렉토리 생성
RUN mkdir -p /app
# /app 디렉토리를 WORKDIR 로 설정
WORKDIR /app
# 현재 Dockerfile 있는 경로의 모든 파일을 /app 에 복사
ADD . /app
# npm install 을 실행
RUN npm install

#환경변수 NODE_ENV 의 값을 development 로 설정
ENV NODE_ENV development

#가상 머신에 오픈할 포트
EXPOSE 3000 80

#컨테이너에서 실행될 명령을 지정
CMD ["npm", "start"]
```
