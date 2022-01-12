---
layout : post
title : "Spring Cloud"
category : Spring
---
### git을 이용해 설정 파일을 저장할 공간을 생성한다

- ${ApplicationName}-${EnvironmentName}.yml 처럼 분리해도되고 1개의 파일에 작성해도된다.
- private 인경우 ssh 설정 추가가 필요하다

### 설정 파일 데이터를 가져올 config server 생성

프로젝트를 생성한 후

```gradle
dependencies {
    implementation('org.springframework.cloud:spring-cloud-config-server')
}
```

application.yml

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/hanbong5938/spring-boot-cloud-yml
          default-label: main
```

main 메서드에 어노테이션 설정

```
@EnableConfigServer
```

실행 후 Request를 보내면 

```http request
GET http://localhost:8080/example/local
```

아래와 같이 Reponse가 온다
```http request

`HTTP/1.1 200 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Fri, 15 Oct 2021 06:18:00 GMT
Keep-Alive: timeout=60
Connection: keep-alive

{
  "name": "example",
  "profiles": [
    "local"
  ],
  "label": null,
  "version": "d98147a844c2a1c8cf326f8185149c7386fbab85",
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/hanbong5938/spring-boot-cloud-yml/Config resource 'file [C:\\Users\\a\\AppData\\Local\\Temp\\config-repo-1891738231702216428\\example.yml' via location '' (document #0)",
      "source": {
        "spring.config.activate.on-profile": "local",
        "who.am.i": "this-local"
      }
    }
  ]
}

Response code: 200; Time: 500ms; Content length: 416 bytes`
```

### Client

클라이언트 프로젝트에 아래 라이브러리를 추가해준다.

```gradle
  implementation('org.springframework.boot:spring-boot-starter-web')
	// https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-config
	implementation("org.springframework.cloud:spring-cloud-starter-config:3.0.5")
	// https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator
	implementation("org.springframework.boot:spring-boot-starter-actuator")
	// https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-context
	implementation("org.springframework.cloud:spring-cloud-context:3.0.4")
	// https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-bootstrap
	implementation("org.springframework.cloud:spring-cloud-starter-bootstrap:3.0.4")
```

bootstrap.yml 을 생성한다. application.yml 보다 먼저 로드된다.(2.4 이후 사용X)

`application.yml` or `bootstrap.yml`
```yaml
server:
  port: 8081

spring:
  application:
    name: example // application 이름
  cloud:
    config:
      uri: http://localhost:8080 // config server port
```

테스트를 위한 controller 생성

```java
@RestController
@RefreshScope
public class ConfigClientController {

    @Value("${who.am.i}")
    private String identity;

    @GetMapping("/test")
    public String test() {
        return identity;
    }
}
```

yml에 있는 값을 읽어오는데 @RefreshScope 어노테이션이 붙은 클래스에는 특정행동 수행시 변경된 설정파일의 설정이 애플리케이션의 재배포과정 없이 실시간으로 반영된다.

**특정행동**수행시 라고 했는데, 설정파일이 변경되면 변경사항을 반영하기위해Config Client에 POST 요청을 하나 날려줘야 한다.

일단 아래와 같이 application.yml 파일을 설정한다.

위와 같이 설정해두면 http://localhost:8081/actuator/refresh 로 POST 요청을 보내면 설정파일을 새로 읽어들여서 애플리케이션이 재기동된다.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: refresh //url 설정
```

client 실행시 profile설정이 git 에 있는 yml 을 가져온다.

client 실행시 첫화면에 아래와 같이 데이터를 가져오는 것을 볼 수 있다.

```log
2021-10-15 16:10:51.160  INFO 23004 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://localhost:8080
2021-10-15 16:10:53.063  INFO 23004 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Located environment: name=example, profiles=[local], label=null, version=b27a4f0c2e3abc3aa9688ae8a91cfb17e0d96d98, state=null
2021-10-15 16:10:53.063  INFO 23004 --- [           main] b.c.PropertySourceBootstrapConfiguration : Located property source: [BootstrapPropertySource {name='bootstrapProperties-configClient'}, BootstrapPropertySource {name='bootstrapProperties-https://github.com/hanbong5938/spring-boot-cloud-yml/Config resource 'file [C:\Users\a\AppData\Local\Temp\config-repo-1891738231702216428\example.yml' via location '' (document #0)'}]
2021-10-15 16:10:53.063  INFO 23004 --- [           main] c.s.cleint.SpringCloudClientApplication  : The following profiles are active: local
```

```http request
# curl -X GET http://localhost:8081/test

GET http://localhost:8081/test
```

이후

git에 있는 yml의 whoami를 변경하고

```http request
# curl -X POST http://localhost:8081/actuator/refresh
POST http://localhost:8081/actuator/refresh
```

다시 test 를 실행시키면 값이 변경된다.

- [spring-boot-cloud-yml](https://github.com/hanbong5938/spring-boot-cloud-yml)
- [spring-boot-cloud-config-server](https://github.com/hanbong5938/spring-boot-cloud-config-server)
- [spring-boot-cloud-client-server](https://github.com/hanbong5938/spring-boot-cloud-client-server)
