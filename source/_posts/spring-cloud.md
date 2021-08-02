---
layout: post
title: Spring Cloud 살펴보기
catalog: true
header-img: 'https://i.imgur.com/avC1Xor.jpg'
tags:
  - Spring
date: 2020-08-01 00:00:00
subtitle:
---


# Spring Cloud Config
스프링 클라우드 컨피그는 중앙 집중식 마이크로서비스 구성을 지원한다. 여기서 두 가지 중요한 구성 요소의 조합이다.

* 스프링 클라우드 컨피그 서버 : 버전 관리 리포지토리로 백업된 중앙 집중식 구성 노출을 지원한다.
* 스프링 클라우드 컨피그 클라이언트 : 애플리케이션이 스프링 클라우드 컨피그 서버에 연결하도록 지원한다.


## GitHub Repository 연결
![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/github-img.png)

GitHub Repository에 
* micoroservice-a-default.yml
* micoroservice-a-dev.yml

아래 처럼 작성합니다.

```yml
application:
  message: "Message From {ENV} Local Git Repository"
```
`{ENV}`에 ddefault, dev 환경에 맞는 값을 작성합니다.


## Config Server 


```gradle
implementation 'org.springframework.cloud:spring-cloud-config-server'
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```
config server 의존 성을 추가합니다. actuator도 편의를 위해서 추가합니다.

```yml
server:
  port: 8888

spring:
  application:
    name: "config-server"

  cloud:
    config:
      server:
        git:
          uri: "https://github.com/cheese10yun/msa-study-sample"

```

config server는 8888 port를 사용하는 관례가 있어 port를 8888로 지정합니다. Github Repository URI 주소를 입력합니다.


```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

  public static void main(String[] args) {
    SpringApplication.run(ConfigServerApplication.class, args);
  }

}
```
`@EnableConfigServer` 어노테이션을 추가만 하면 별다른 설정없이 Config Server가 설정됩니다. 

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/intellj-actuator.png)
actuator 의존성이 있으면 IntellJ Endpoints -> Mappings에서 현재 서버의 API mapping 정보를 쉽게 확인할 수 있습니다.


마우스 왼쪽 클릭을 한 이후에 Open In Http Request...를 클릭하면 쉽게 HTTP 콜을 할 수 있습니다.

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/http-call.png)

URL 형식은 /{appliation-name}/{profile}/{label}입니다. 위에서 등록한 `micoroservice-a-default.yml`을 확인해보기 위해서 `http://localhost:8888/microservice-a/default`을 호출합니다.


```json
{
  "name": "microservice-a",
  "profiles": [
    "default"
  ],
  "label": null,
  "version": "c03eecc5d8eabefc4b2a8f085789f42bd5317366",
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/cheese10yun/msa-study-sample/microservice-a-default.yml",
      "source": {
        "application.message": "Message From Default Local Git Repository"
      }
    }
  ]
}
```
응답 값을 보면 해당 properties를 잘 읽어 오는 것을 확인할 수 있습니다. 

`http://localhost:8888/microservice-a/dev`을 호출하면 `micoroservice-a-dev.yml`의 값을 제대로 읽어 오는지 확인할 수 있습니다.

```json
{
  "name": "microservice-a",
  "profiles": [
    "dev"
  ],
  "label": null,
  "version": "c03eecc5d8eabefc4b2a8f085789f42bd5317366",
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/cheese10yun/msa-study-sample/microservice-a-dev.yml",
      "source": {
        "application.message": "Message From Default Dev Git Repository"
      }
    }
  ]
}
```

## Client

```gradle
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-config-client'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```
필요한 의존성을 추가합니다.


```java
@Component
@ConfigurationProperties("application")
@Getter
@Setter
public class ApplicationConfiguration {

  private String message;

}

```
프로퍼티를 읽을 ConfigurationProperties 객체를 만듭니다.


```java
@RestController
@RequiredArgsConstructor
public class WelcomeController {

  private final ApplicationConfiguration applicationConfiguration;


  @GetMapping("/message")
  public Map<String, String> welcome() {

    final Map<String, String> map = new HashMap<>();
    map.put("message", applicationConfiguration.getMessage());

    return map;
  }
}
```
해당 메시지를 확인할 수 있는 컨트롤러를 만듭니다. `getMessage()` 메시지는 각 환경마다 다른 메시지를 출력합니다.

```yml
spring:

  profiles:
    active: {ENV}

  cloud:
    config:
      uri: http://localhost:8888
  application:
    name: microservice-a

```
파일명은 `bootstrap.yml`으로 지정합니다. `active`는 각 환경마다 지정합니다. dev으로 지정하고 `http://127.0.0.1:8080/message` API를 호출해보겠습니다.

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/message-call.png)

이것도 인텔리제이를 이용해서 호출하면 간편합니다.


```json
{
  "message": "Message From Default Dev Git Repository"
}
```

`Dev` 메시지가 출력되는 것을 확인 할 수 있습니다. `profiles`을 `default` 으로 지정하면 메시지에는 local 이라는 문자가 출력됩니다.


## Refresh
마이크로서비스 A의 properties의 변경이 생겼을 경우 Refresh API를 호출해서 설정을 변경할 수 있다.

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/json-1.png)

message API를 호출하면 기존 메시지가 그대로 출력된다.


```yml
# microservice-a-default.yml
application:
  message: "Message From Default Local Git Repository (Properties update...)"
```
`microservice-a-default.yml` 메시지를 위와 같이 변경한 이후에 


POST http://127.0.0.1:8080/actuator/refresh을 호출하면

```
[
  "config.client.version",
  "application.message"
]
```

Response으로 application.message의 변경을 알려준다. 이후 message api를 호출하면 아래 그림처럼 변경된 Response를 확인 할 수 있다.

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/json-2.png)



## 정리
Github Repository와 같은 버전 관리 리포지토리로 백업된 중앙 집중 구성으로 properties를 제공해줍니다. 또 Properties 설정 및 단순한 설정으로 클라이언트 애플리케이션이 스프링 클라우드 컨피그 서버에 쉽게 연결되도록 지원해줍니다.

# Spring Cloud Bus

프로덕션 환경에서 실행중인 마이크로서비스 A의 인스턴스가 N대가 있을 경우에는 설정을 변경하기 위해서는 `POST http://127.0.0.1:8080/actuator/refresh` 요청을 N대의 인스턴스 서버에 N번의 refresh API를 호출 해야한다. 이는 번거러울 뿐만 아니라 실수를 유발하기 쉬운 구조이다.


## 스프링 클라우드 버스를 시용한 변경 전파

스프링 클라우 버스를 사용해서 Rabbit MQ 같은 메시지 브로커를 통해 변경 사항을 여러 인스턴스에 전달할 수 있다.

각 마이크로서비스의 인스턴스는 애플리케이션 구동시에 스프링 클라우드 버스에 등록한다. 마이크로서비스의 인스턴스 중 하나에 refresh가 호출되면 스프링 클라우드 버스는 모든 마이크로서비스 인스턴스에 변경 이벤트를 전달하게 된다. 


## Rabbit MQ 설치

```
.
├── config-server
├── docker-compose.yaml
├── microservice-a
├── microservice-a-default.yml
├── microservice-a-dev.yml
├── service-consumer
├── static
└── volumes
```
프로젝트 루트 디렉토리에 volumes 디렉토리를 만들고, gitginore를 추가 해줍니다. 해당 디렉토리에 docker rabbmit mq가 저장됩니다.


`docker-compose.yaml`  파일은 아래와 같이 준비 해줍니다.

```yaml
version: "3"

services:
  rabbitmq:
    container_name: bus.rabbitmq
    image: rabbitmq:3.7-management
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      - RABBITMQ_DEFAULT_USER = user
      - RABBITMQ_DEFAULT_PASS = user
    hostname: bus
    volumes:
      - ./volumes/bus-rabbitmq:/var/lib/rabbitmq
```


```
$ cd 프로젝트 루트 디렉토리
$ docker-compose -up -d
```

## Client
```gradle
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp'
}
```
필요한 의존성을 추가합니다.

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/bus-log.png)
서버가 시작될때 스프링 클라우드 버스에 등록되고, 클라우드 버스의 이벤트를 수신하게 됩니다. Rabbmit MQ 커넥션은 자동으로 연결됩니다. (스프링 너란 자식....)


![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/intellj-port-2.png)

포트를 8080, 8081을 설정해서 2대의 서버를 구동 시킵니다.



## Bus Refresh
![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/bus-mapping.png)

`microservice-a-default.yml`을 아래처럼 변경합니다.

```yml
application:
  message: "Message From Default Local Git Repository (Spring Cloud Bus)"
```

`/bus/refresh` API가 추가되 었습니다. 해당 API를 호출합니다. (8081을 호출해도 됩니다.)

```curl
curl -X POST http://localhost:8080/bus/refresh
```

refresh 호출 이후에 message를 호출합니다.


```curl
curl -X GET http://localhost:8081/message
```
8080 refresh를 호출하더라도 다른 8081 서버도 반영되는 것을 확인할 수 있습니다.

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/8081-result.png)

# Eureka

## 네임 서버
마이크로서비스 아키텍처는 서로 상호 작용하는 더 작은 마이크로서비스가 필요 하다. 이 밖에도 각 마이크로서비스의 인스턴스가 여러 개 있을 수 있다. 마이크로서비스의 새로운 인스턴스가 동적으로 생성되고 파괴되면 외부 서비스의 연결 및 구성을 수동으로 유지하는 것이 어려울 수 있다. **네임 서버는 서비스 등록 및 서비스 검색 기능을 제공한다.** 네임 서버는 마이크서비스가 이들 자신을 등록할 수 있게 하고, 상호 작용하고자 하는 다른 마이크러서비스에 대한 URL을 찾을 수 있게 도와준다.

## URL 하드 코딩의 한계

```yml
microservice-a:
  ribbon:
    listOfServers: http://localhost:8080,http://localhost:8081
```
* 마이크로서비스 A의 새 인스턴스가 생성된다.
* 마이크로서비스 A의 기존 인스턴스는 더 이상 사용할 수 없다.
* 마이크로서비스 A가 다른 서버로 이동됐다.

이런 모든 경우에 구성을 업데이트해야 하며, 변경 사항을 적용하기 위해서는 마이크로서비스가 새로 고쳐져야 한다.

## 네임 서버 작동

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/name-server.png)

* **모든 마이크로서비스는 각 마이크로서비스가 시작될때 네임 서버에 등록한다.**
* 서비스 소비자가 특정 마이크로 서비스의 위치를 얻으려면 네임 서버를 요청해야한다.
* 고유한 마이크로서비스 ID가 각 마이크로서비스에 지정된다. 이것을 등록 요청 및 검색 요청에서 키로 사용된다.
* 마이크로서비스는 자동으로 등록 및 등록 취소할 수 있다.
* 서비스 소비자가 마이크로서비스ID로 네임 서버를 찾을 때마다 해당 특정 마이크로서비스의 인스턴스 목록을 가져온다.



## 구현

### 유레카 서버 설정


```
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
}
```
* 필요한 디펜던시를 추가합니다.

```java

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

  public static void main(String[] args) {
    SpringApplication.run(EurekaServerApplication.class, args);
  }
}
```

```yml
server:
  port: 8761

eureka:
  client:
    fetch-registry: false
    register-with-eureka: false
```

### 유레카 마이크로 서비스 등록
기존에 서비스들을 유레카 서버에게 접속할 수있도록 서비스 등록 작업을 진행합니다.

```
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
}
```

```java
@SpringBootApplication
@EnableDiscoveryClient
public class MicroserviceAApplication {

  public static void main(String[] args) {
    SpringApplication.run(MicroserviceAApplication.class, args);
  }

}
```
의존성 추가 및 `@EnableDiscoveryClient` 어노테이션 추가합니다. 

### 유레카와 마이크로 서비스 연결


```yml
#microservice-a:
#  ribbon:
#    listOfServers: http://localhost:8080,http://localhost:8081
```

```java
@SpringBootApplication
@EnableFeignClients("com.cloud.serviceconsumer") // 어노테이션 추가
@EnableDiscoveryClient
public class ServiceConsumerApplication {

  public static void main(String[] args) {
    SpringApplication.run(ServiceConsumerApplication.class, args);
  }

}
```

마이크로서비스A의 URL을 하드 코딩했을 경우 추가적인 서버 증성작업이 어렵다. 서비스 소비자 마이크로서비스가 유레카 서비스로부터 URL을 알아낼 수 있어야한다. 하드코딩된 URL정보를 주석처리 하고 아래의 어노테이션을 추가하면 된다.


```java
@FeignClient(name = "microservice-a")
@RibbonClient(name = "microservice-a")
public interface RandomServiceProxy {

  @GetMapping("/random")
  List<Integer> getRandomNumbers();
}
```

### 동작 순서
1. 마이크로서비스 A의 각 인스턴스가 시작되면 유레카 네임 서버에 등록한다.
2. 서비스 소비자 마이크로서비스는 마이크로서비스 A의 인스턴스에 대해 유레카 네임 서버를 요청한다.
3. 서비스 소비자 마이크로서비스는 립본 클라이언트-클라이언트 로드 밸런서를 사용해 소출할 마이크로서비스 A의 특정 인스턴스를 결정한다.
4. 서비스 소비자 마이크로서비스는 마이크로서비스 A의 특정 인스턴스를 호출한다.

유레카의 가장 큰 장점은 서비스 소비자 마이크로서비스가 마이크로서비스 A와 분리된다는 것이다. 서비스 소비자 마이크로서비스는 마이크로서비스 A의 새로운 인스턴스가 나타나거나 기존 인스턴스가 디운될 때마다 재구성할 필요가 없다.


![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/eureka-dashboard.png)
유레카 데시보드에서 여러 마이크로서비스를 확인 할 수 있다.

# Ribbon

## 로드 밸런싱
마이크로서비스는 클라우드-네이티브 아키텍처의 가장 중요한 빌딩 블록이다. 마이크로서비스 인스턴스는 특정 마이크로서비스의 로드에 따라 확대 및 축소된다. 부하가 마이크로서비스의 다른 인스턴스 간에 똑같이 분산되도록 하려면 로드밸런싱의 기술피 필수이다. 로드 밸런싱은 로드가 마이크로서비스의 다른 인스턴스간에 균등하게 분배하도록 도와준다.

## Ribbon 구성
![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/ribbon.png)

스프링 클라우드 넷플릭스 립본은 마이크로서비스의 다른 인스턴스 간에 라운드 로빈 실행을 사용해 **클라이언트-사이드 로드 밸런싱을 제공한다.**

```gradle
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-ribbon'
}
```
필요한 의존성을 추가 합니다.

```java
@FeignClient(name = "microservice-a")
@RibbonClient(name = "microservice-a")
public interface RandomServiceProxy {

  @GetMapping("/random")
  List<Integer> getRandomNumbers();

}

```
기존에 작성했던 RandomServiceProxy 인터페이스를 위와 같이 작성합니다. `FeignClient`도 서비스 네임만 기술합니다.

```yml
microservice-a:
  ribbon:
    listOfServers: http://localhost:8080,http://localhost:8081
```
`microservice-a`의 서비스 url을 입력합니다.

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/ribbon-result.png)

GET http://127.0.0.1:8100/add을 호추랗면 아래와 같은 결과값을 응답 받습니다.



![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/ribbon-result2.png)

라운드 로빈 방식으로 random API 호출이 http://localhost:8080, http://localhost:8081으로 분산되어 호출됩니다.

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/msa-simple.png)

위과 같은 아키텍처 구성을 가지게 됩니다.

# 선언적 Rest 클라이언트 - Feign

페인은 최소한의 구성과 코드로, REST 서비스를 위한 REST 클라이언트를 쉽게 작성할 수 있습니다. 간단한 인터페이스로, 적절한 어노테이션을 사용하는 것이 특징입니다. 

페인은 립본 및 유레카와 통합하여 사용하면 더욱 효율성이 높아지게 됩니다.


```gradle
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
}
```
필요한 의존성을 추가합니다.

```java
@FeignClient(name = "micoservice-a", url = "localhost:8080")
public interface RandomServiceProxy {

  @GetMapping("/random")
  public List<Integer> getRandomNumbers();
}
```
* 서비스의 이름과 URL을 하드코딩 합니다. (유레카를 통해서 하드코딩된 부분을 제거할 수 있습니다.)
* Controller 코드를 작성하듯이 작성합니다.
* 중요한 것은 이것은 인터페이스이며, 적절한 어노테이션 기반으로 동작한다는 것입니다.


```java

@RestController
@Slf4j
@RequiredArgsConstructor
public class NumberAddController {

  private final RandomServiceProxy randomServiceProxy;

  @GetMapping("/add")
  public Long add() {
    final List<Integer> numbers = randomServiceProxy.getRandomNumbers();
    final long sum = numbers.stream().mapToInt(number -> number).asLongStream().sum();
    log.warn("returning " + sum);
    return sum;
  }
}

```
* RandomServiceProxy 의존성을 받아 사용합니다.

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/fegin-result.png)
Feign 응답값을 확인할 수 있습니다.

# API 게이트웨이

## 마이크로 서비스의 문제점 
* 인증, 권한 부여 및 보안 : 모놀리식 서버에서는 인증 인가를 한 서버에서 책임지면 되는데 여러 서비스가 존재하는 마이크로서비스에서는 올바른 엑세스 보장을 어떻게할 것인가?
* 동적 라우팅은 어떻게 할 것인가?
* 내결합성: 하나의 마이크로서비스에서 오류가 발생해도 전체 시스템이 중단되지 않도록하면 어떻게 해야하는가 ?

이러한 문제는 마이크로서비스가 서로 직접 대화할 때 개별 마이크로서비스에 의해 해결돼야한다. **하지만 이런 아키텍처들은 위 우려들을 각 마이크로서비스가 다르게 처리할 수 있기 때문에 유지 관리가 어려울 수 있다.**

이려한 문제를 해결하기 가장 쉬운 것이 API 게이트웨이이다. 다음은 API 게이트가 제공하는 기능이다.

* 인증 및 보안
* 속도 제한
* 모니터링
* 동적 라우팅 및 정적 응답 처리
* 로드 차단
* 여러 가지 서비스의 응답 집계


## 주울로 클라이언트 - 사이드 로드 밸런싱 구현
주을은 스프링 클라우드 넷플릭스 프로젝트의 일부다. 동적 라우팅, 모니터링, 필터링, 보안 등의 기능을 제공하는 API 게이트웨이 서비스다.

```
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-zuul'
}
```

```java
@EnableZuulProxy
@EnableDiscoveryClient
@SpringBootApplication
public class ApiGatewayApplication {

  public static void main(String[] args) {
    SpringApplication.run(ApiGatewayApplication.class, args);
  }

}
```

```yml
spring:
  application:
    name: "zuul-api-gateway"


eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka

server:
  port: 8765

```

필요한 의존성, 유레카 클라이언트, API 게이트웨이 등록을 진행한다.

![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/api-gateway-eureka.png)
서버를 실행하면 유레카에 등록된 것을 확인 할 수있다.

```java
@Component
@Slf4j
public class SimpleLoggingFilter extends ZuulFilter {
  
  @Override
  public String filterType() {
    return "pre"; // pre 사전 필터, route 원본으로 라우팅하기, post필터를 포스트라우팅, error 에러 처리
  }

  @Override
  public int filterOrder() {
    return 1; // 필터 우선순위
  }


  @Override
  public boolean shouldFilter() {
    return true; // 조건에 따른 필터 동작 여부, 해당 코드에서는 동작하기 위해서 true 리턴
  }
  
  @Override
  public Object run() { // 필터에 대한 로직을 구현하는 방식, 아래 예제는 단순하게 로깅용
    final RequestContext context = RequestContext.getCurrentContext();
    final HttpServletRequest request = context.getRequest();

    log.info(MessageFormat
        .format("Request Method : {0} \n URL : {1} ", request.getMethod(),
            request.getRequestURI()));
    return null;
  }
}
```


```
GET http://127.0.0.1:8765/microservice-a/random
Accept: application/json

GET http://127.0.0.1:8765/service-consumer/add
Accept: application/json
```
service-name/url 을 호출할 경우 유레카에 등록되있는 서비스 정보를 기반으로 호출하게된다.
![](https://github.com/cheese10yun/msa-study-sample/raw/master/static/api-gateway-eureka-result.png)

`SimpleLoggingFilter` 잘 동작하는지 확인할 수 있다.


## 필터
* 사전 필터: 주울에서 목표 대상에 대한 실제 요청이 발생하기 전에 호출된다. 일반적으로 사전 필터는 서비스의 일관된 메세지형식(HTTP 헤더의 포함여부)을 확이하는 작업을 수행하거나 서비스를 이용하는 사용자가 인증 및 인가되었는지 확인하는 게이이트키퍼 역할을 한다.
* 사후 필터: 대상 서비스를 호추랗고 응답을 클라이언트로 전송한 후 호출된다. 일반적으로 사후 필터는 대상 서비스의 응답을 로깅하거나 에러 처리, 민감정보에 대한 응답을 감시하는 목적으로 구현된다.
* 경로 필터: 대상 서비스가 호출되기 전에 호출을 가로체는 데 사용된다. 일반적으로 경로 필터는 일정 수준의 동적 라우팅 필요 여부를 결정하는 데 사용된다. 예를 들어 이 장 뒷부분에서 동일 서비스의 다른 두 버전을 라우팅할 수 있는 경로 단위 핉를 사용해 작은 호출 비율만 새 버전의 서비스로 라우팅할 수 있다.

### 필터 적용 패턴
* TrackingFilter: 주울에서 보내는 모든 요청에 연관된 상관관계 ID 여부를 확인하는 사전 필터다, 상관관계 ID는 고객 요청을 수행할 때 실행할 때 실행되는 모든 마이크로서비스에 전달되는 고유 ID이다. 상관관계 ID를 사용하면 특정 호출이 일련의 마이크로서비스를 통과할 때 발생하는 모든 이벤트 체인을 추적할 수 있다.
* SpecialRoutesFilter: 유입되는 경로를 확인하고 해당 경로에 A/B 테스팅 수행 여부를 결정하는 주울의 경로 필터다. 
* ResponseFilter: 서비스 호출과 연관된 상관관계 ID를 클라이언트 회신하는 HTTP 응답 헤더에 삽입하는 사후 필터다. 일반적으로 클라이언트는 호출한 요청과 연관된 상관관계 ID에 엑세스할 수 있다.


## 참고
* [스프링 5.0 마스터](http://acornpub.co.kr/book/mastering-spring-5.0)
* [스프링 마이크로 서비스 코딩 공작소](http://www.yes24.com/Product/Goods/67473377)