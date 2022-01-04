---
layout : post
title : "MariaDB(Compose포함)옵션"
category : docker
---
```bash
docker pull mariadb
```

```bash
docker container run --restart=always -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=비밀번호 --name 이름 mariadb
```

외부포트 : 내부포트

```bash
// 볼륨 설정시
docker container run -d -p 3306:3306 	\
-e MYSQL_ROOT_PASSWORD=1234 		\
-v /Users/Shared/happy/mariadb:/var/lib/mysql 	\
--name mariadb mariadb
```

- `v`는 Bind mount a volume와 관련한 옵션이다.

```
-v <bind-path>:<file or directory on the host machine>
```

그리고 `--name` 옵션은 컨테이너 실행시 컨네이너 실행시 컨테이너 이름을 부여하기 위한 용도이다.

```
--name <original-container-name> <new-container-name>
```

접속

```bash
docker exec -i -t mariadb bash
```

재시작

```bash
#만들때 --restart-always 옵션을 넣어주면 되는데, 빼고 컨테이너를 실행했을 때에는 아래의 명령으로 변경할 수 있다.

docker update --restart=always <container-id>

```

### Compose 사용시

- run_tests 파일 작성
```docker
version: '3.1'

services:
  db:
    container_name: mariadb
    image: mariadb:latest
    restart: always
    ports:
      - 53306:3306
    volumes:
      - /docker/con_volumes/mariadb/data:/var/lib/mysql
      - /docker/con_volumes/mariadb/conf.d:/etc/mysql/conf.d
    environment:
      MYSQL_ROOT_PASSWORD: fdsa1234
```

- 시작
```shell
docker-compose up -d ./run_tests
```

- 중단
```shell
docker-compose down
```

