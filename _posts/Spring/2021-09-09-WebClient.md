---
layout : post
title : "Web Client"
category : Spring
---


```gradle
// https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-webflux
    implementation 'org.springframework.boot:spring-boot-starter-webflux:2.5.4'
```

WebFlux 설치 이후

```java
@Component
public class BizApi {

    private final String API_SERVICE_KEY = "";

    private final WebClient webClient;

    public BizApi(WebClient.Builder builder) {
        this.webClient = builder.baseUrl("https://api.odcloud.kr/api/nts-businessman/v1")
                .defaultHeader("Content-Type", "application/json;charset=utf-8")
                .defaultHeader("accept", "application/json;charset=utf-8")
                .build();
    }

    public Mono<BizDto.Res> isActive(BizDto.Req bizRequest) {
        return webClient.post()
//                .uri("?serviceKey=" + API_SERVICE_KEY)
                .uri(uriBuilder -> UriComponentsBuilder.fromUri(uriBuilder.build())
                .path("/status")
                .queryParam("serviceKey", "{serviceKey}")
                .encode()
                .buildAndExpand(API_SERVICE_KEY)
                .toUri())
                .body(Mono.just(bizRequest), BizDto.Req.class)
                .retrieve()
                .bodyToMono(BizDto.Res.class)
								.retryWhen(Retry.max(5));
    }
}
```

기본적으로 uri로 사용하면되나

uri에 + 기호가 들어가있는 경우 인코딩이 되지 않는다

UriBuilder를 사용하여 처리한다

Mono는 0~1개 Flux는 다수의 데이터 처리시 사용한다.

retryWhen 을 이용하여 재시도 횟수 설정 가능하다.

```java
Object data = webClient.post()
                .uri(uriBuilder -> UriComponentsBuilder.fromUri(uriBuilder.build())
                .path("/status")
                .queryParam("serviceKey", "{serviceKey}")
                .encode()
                .buildAndExpand(API_SERVICE_KEY)
                .toUri())
                .body(Mono.just(bizRequest), BizDto.Req.class)
                .retrieve()
                .bodyToMono(BizDto.Res.class)
								.flux()
		            .toStream()
		            .findFirst()
		            .orElse(defaultValue);
```

Mono 같은경위 위처럼 데이터를 가져오고

```java
List<SomeData> results =
    webClient.mutate()
             .baseUrl("https://some.com/api")
             .build()
             .get()
             .uri("/resource")
             .accept(MediaType.APPLICATION_JSON)
             .retrieve()
             .bodyToFlux(SomeData.class)
             .toStream()
             .collect(Collectors.toList());
```

Flux는 위와 같이 처리한다.


Spring에서 주로 사용하던 RestTemplate는 5.0 버전부터 deprecated 되었다.

WebClient 사용하기를 권고하고 있으며 아래의 특징을 가지고 있다.
```
- Non-blocking I/O
- Reactive Streams back pressure
- High concurrency with fewer hardware resources
- Functional-style, fluent API that takes advantage of Java 8 lambdas
- Synchronous and asynchronous interactions
- Streaming up to or streaming down from a server
```

# Static factory 사용

가장 간편하게 사용할 수 있는 방법이다.
`@Configuration`을 사용하여 여러 빈에서 사용할 수 있도록 한다.

```
WebClient.create();
WebClient.create(String baseUrl);
```

그러나 header, cookie, default, filter, ConnectionTimeOut 등의 설정을 원하는 경우 `Builder`를 사용하는 것이 좋다.

```
- 모든 호출에 대한 기본 Header / Cookie 값 설정
- `filter`를 통한 Request/Response 처리
- Http 메시지 Reader/Writer 조작
- Http Client Library 설정
```


## MaxInMemorySize

`Spring WebFlux`는 메모리 문제를 피하기위하여 codec 처리를 위한 in-memory buffer의 값이 256KB로 설정되어있다.
이보다 큰 HTTP 메세지 처리시에는 `DataBufferLimitException`이 발생하게 되며 메모리 사이즈를 늘려주기 위해서는 `ExchageStrategies.builder()`를 사용한다.


```java
ExchangeStrategies exchangeStrategies =
    ExchangeStrategies
        .builder()
        .codecs(configurer->configurer.defaultCodecs()
                                     .maxInMemorySize(1024*1024*1))
        .build();
```

## Logging

`Debug`레벨에서 `formData`, `Trace`레벨에서 `header` 정보는 보안적인 문제가 있을 수 있어 로그에서 확인할 수 없게 설정되어있다.
로그확인이 필요한 경우 `ExchageStrateges`, logging level을 아래와 같이 설정한다.
```
exchangeStrategies
    .messageWriters().stream()
    .filter(LoggingCodecSupport.class::isInstance)
    .forEach(writer->((LoggingCodecSupport)writer).setEnableLoggingRequestDetails(true));
```

```
logging:
  level:
    org.springframework.web.reactive.function.client.ExchangeFunctions: DEBUG
```

## Client Filters

Request나 Response를 수정하기 위해서 `WebClient.builder().filter()`를 사용한다.

아래와 같이 수정하여 사용한다.
```
WebClient.builder()
        .filter(ExchangeFilterFunction.ofRequestProcessor(
            clientRequest->{
                log.debug("Request: {} {}", clientRequest.method(), clientRequest.url());
                clientRequest.headers()
                             .forEach((name, values)->values.forEach(value->log.debug("{} : {}", name, value)));
                return Mono.just(clientRequest);
            }
        ))
        .filter(ExchangeFilterFunction.ofResponseProcessor(
            clientResponse->{
                clientResponse.headers()
                              .asHttpHeaders()
                              .forEach((name, values)->
values.forEach(value->log.debug("{} : {}", name, value)));
                return Mono.just(clientResponse);
            }
        ))
```

## HttpClient TimeOut

`HttpClient`, `ConnectionTimeOut`같은 설정값을 변경하기 위해서는 `WebClient.builder().clientConnector()`를 사용하여 변경해주어야 한다.
```
WebClient
  .builder()
    .clientConnector(
      new ReactorClientHttpConnector(
        HttpClient
          .create()
            .secure(
              ThrowingConsumer.unchecked(
                sslContextSpec->sslContextSpec.sslContext(
                  SslContextBuilder
                    .forClient()
                 .trustManager(InsecureTrustManagerFactory.INSTANCE)
                    .build()
                )
              )
            )
            .tcpConfiguration(
              client->client.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 120_000)
      .doOnConnected(
        conn->conn.addHandlerLast(new ReadTimeoutHandler(180))
                    .addHandlerLast(new WriteTimeoutHandler(180))
                    )
            )
      )
    )
```

`new ReactorClientHttpConnector()`를 통해 옵션이 추가된 새로운`HttpClient`를 설정한다.

위 예제에서는 `HTTPS` 인증서를 검증하지 않고 바로 접속하는 설정과, `TCP` 연결 시 `ConnectionTimeOut` , `ReadTimeOut` , `WriteTimeOut` 을 적용하는 설정을 추가하였다.

# **Usage**

`WebClient` 는 기존 설정값을 상속해서 사용할 수 있는 `mutate()`가 있다.
`mutate()` 를 통해 `builder()` 를 다시 생성하여 추가적인 옵션을 설정하여 재사용이 가능하기 때문에 `@Bean` 으로 등록한 `WebClient`는 각 `Component` 에서 의존주입하여 `mutate()`를 통해 사용 하는 것이 좋다.

```
WebClient a = WebClient.builder()
                       .baseUrl("https://some.com")
                       .build();
WebClient b = a.mutate()
               .defaultHeader("user-agent", "WebClient")
               .build();
WebClient c = b.mutate()
               .defaultHeader(HttpHeaders.AUTHORIZATION, token)
               .build();
```

위와 같이 설정 했을 경우 `WebClient` “c”는 “a”와 “b”에 설정된 `baseUrl` , `user-agent` 해더를 모두 가지고 있습니다.

`@Bean` 으로 등록된 `WebClient` 는 다음과 같이 사용 가능합니다.

```
@Service
@RequiredArgsConstructor
@Slf4j
public class SomeService implements SomeInterface {

    private final WebClient webClient;    public Mono<SomeData> getSomething() {

    returnwebClient.mutate()
                    .build()
                    .get()
                    .uri("/resource")
                    .retrieve()
                    .bodyToMono(SomeData.class);
    }
}
```

## **retrieve() vs exchange()**

`HTTP` 호출 결과를 가져오는 두 가지 방법으로 `retrieve()` 와 `exchange()` 가 존재합니다. `retrieve` 를 이용하면 바로 ResponseBody를 처리 할 수 있고, `exchange` 를 이용하면 세세한 컨트롤이 가능합니다. 하지만 Spring에서는 `exchange` 를 이용하게 되면 Response 컨텐츠에 대한 모든 처리를 직접 하면서 발생할 수 있는 memory leak 가능성 때문에 가급적 `retrieve` 를 사용하기를 [권고](https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/web-reactive.html#webflux-client-exchange)하고 있다.

- `retrieve`

```
Mono<Person> result = webClient.get()
                               .uri("/persons/{id}", id)
                               .accept(MediaType.APPLICATION_JSON)
                               .retrieve()
                               .bodyToMono(Person.class);
```

- `exchange`

```
Mono<Person> result = webClient.get()
                               .uri("/persons/{id}", id)
                               .accept(MediaType.APPLICATION_JSON)
                               .exchange()
                               .flatMap(response ->
                                 response.bodyToMono(Person.class));
```

## **4xx and 5xx 처리**

`HTTP` 응답 코드가 4xx 또는 5xx로 내려올 경우 `WebClient` 에서는 `WebClientResponseException`이 발생하게 됩니다. 이 때 각 상태코드에 따라 임의의 처리를 하거나 `Exception` 을 랩핑하고 싶을 때는 `onStatus()` 함수를 사용하여 해결 할 수 있다.

```
webClient.mutate()
         .baseUrl("https://some.com")
         .build()
         .get()
         .uri("/resource")
         .accept(MediaType.APPLICATION_JSON)
         .retrieve()
         .onStatus(status->status.is4xxClientError()
                          || status.is5xxServerError()
             , clientResponse->
clientResponse.bodyToMono(String.class)
                           .map(body->new RuntimeException(body)))
         .bodyToMono(SomeData.class)
```

## **GET**

`GET` 호출은 앞서 보여진 예시에서 처럼 `get()` 함수를 통해 사용되며 `uri()` 를 통해 호출 리소스 정보를 전달해 줘야 한다. 만약 Query 파라미터가 존재한다면 다음과 같이 변수를 추가 줄 수 있다.

```
public Mono<SomeData> getData(Integer id, String accessToken) {
    return
        webClient.mutate()
                 .baseUrl("https://some.com/api")
                 .build()
                 .get()
                 .uri("/resource?id={ID}", id)
                 .accept(MediaType.APPLICATION_JSON)
                 .header(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken)
                 .retrieve()
                 .bodyToMono(SomeData.class)
        ;
}
```

`uri()` 함수를 제공하는 `UriSpec` 인터페이스는 아래와 같으며, `Map` 을 이용하거나 직접 `UriBuilder` 등을 통해 세세한 컨트롤도 가능하다.

## **POST**

Body Contents를 전송할 수 있는 `POST` 호출 은 `post()` 함수를 통해 제공되며, RequestBody를 설정하기 위한 `RequestBodySpec` 인터페이스는 다음과 같다.

- form 데이터 전송

```
webClient.mutate()
         .baseUrl("https://some.com/api")
         .build()
         .post()
         .uri("/login")
         .contentType(MediaType.APPLICATION_FORM_URLENCODED)
         .accept(MediaType.APPLICATION_JSON)
         .body(BodyInserters.fromFormData("id", idValue)
                            .with("pwd", pwdValue)
         )
         .retrieve()
         .bodyToMono(SomeData.class);
```

form 데이터를 생성하기 위해서는 `BodyInserters.fromFormData()` 를 이용할 수 있으며, `bodyValue(MultiValueMap<String, String>)` 을 통해서도 데이터를 전송 할 수 있습니다.

- JSON body 데이터 전송

```
webClient.mutate()
         .baseUrl("https://some.com/api")
         .build()
         .post()
         .uri("/login")
         .contentType(MediaType.APPLICATION_JSON)
         .accept(MediaType.APPLICATION_JSON)
         .bodyValue(loginInfo)
         .retrieve()
         .bodyToMono(SomeData.class);
```

객체 자체를 RequestBody로 전달하기 위해서는 `bodyValue(Object body)` 를 사용하거나 `body(Object producer, Class<?> elementClass)` 를 통해서 사용할 수 있다.

또한 `Mono` 나 `Flux` 객체를 통해 RequestBody를 생성하기 위한

```
<T, P extends Publisher<T>> RequestHeadersSpec<?> body(P publisher, Class<T> elementClass);
```

함수도 존재한다.

## **PUT**

`PUT` 호출은 `POST` 호출과 유사하다.

```
webClient.mutate()
         .baseUrl("https://some.com/api")
         .build()
         .put()
         .uri("/resource/{ID}", id)
         .contentType(MediaType.APPLICATION_JSON)
         .accept(MediaType.APPLICATION_JSON)
         .bodyValue(someData)
         .retrieve()
         .bodyToMono(SomeData.class);
```

## **DELETE**

`DELETE` 호출은 `GET` 과 유사하며 `delete()` 함수를 통해 시작되고, `delete()` 함수 의 특성상 response는 `Void.class` 로 처리된다.

```
webClient.mutate()
         .baseUrl("https://some.com/api")
         .build()
         .delete()
         .uri("/resource/{ID}", id)
         .retrieve()
         .bodyToMono(Void.class);
```

# **Synchronous Use**

`WebClient` 는 `Reactive Stream` 기반이므로 리턴값을 `Mono` 또는 `Flux` 로 전달받게 됩니다. Spring WebFlux를 이미 사용하고 있다면 문제가 없지만 Spring MVC를 사용하는 상황에서 `WebClient` 를 활용하고자 한다면 `Mono` 나 `Flux` 를 객체로 변환하거나 `Java Stream` 으로 변환해야 할 필요가 있다.

이럴 경우를 대비해서 `Mono.block()` 이나 `Flux.blockFirst()` 와 같은 blocking 함수가 존재하나 `block()` 을 이용해서 객체로 변환하면 `Reactive Pipeline` 을 사용하는 장점이 없어지고 모든 호출이 main 쓰레드에서 호출되기 때문에 Spring 측에서는 `block()` 은 테스트 용도 외에는 가급적 사용하지 말라고 권고하고 있다.

대신 완벽한 Reactive 호출은 아니지만 `Lazy Subscribe` 를 통한 `Stream` 또는 `Iterable` 로 변환 시킬 수 있는 `Flux.toStream()` , `Flux.toIterable()` 함수를 제공하고 있다.

```
List<SomeData> results =
webClient.mutate()
             .baseUrl("https://some.com/api")
             .build()
             .get()
             .uri("/resource")
             .accept(MediaType.APPLICATION_JSON)
             .retrieve()
             .bodyToFlux(SomeData.class)
             .toStream()
             .collect(Collectors.toList());
```

`Flux.toStream()` 을 통해 데이터를 추가 처리하거나 List로 변환하여 사용할 수 있다.

`Mono` 에 대해서는

```
SomeData data =
webClient.mutate()
             .baseUrl("https://some.com/api")
             .build()
             .get()
             .uri("/resource/{ID}", id)
             .accept(MediaType.APPLICATION_JSON)
             .retrieve()
             .bodyToMono(SomeData.class)
             .flux()
             .toStream()
             .findFirst()
             .orElse(defaultValue);
```

`Mono.flux()` 를 통해 `Flux` 로 변환하고 `findFirst()` 를 통해 `Optional` 처리 하는 것이 좋다. (이 때는 onError 처리가 필요)