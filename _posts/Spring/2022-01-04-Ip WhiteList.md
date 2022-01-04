---
layout : post
title : "Ip White List 확인하기"
category : Spring
---
## Dependency

- **`IpAddressMatcher`를 사용하는 경우 필요**

```
implementation 'org.springframework.security:spring-security-web'
```

spring-security-web 의존성을 추가한다.

## Interceptor 생성 및 등록

```java
@RequiredArgsConstructor
@Component
public class IpAddressAccessControlInterceptor implements HandlerInterceptor {

    @Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)throws Exception {
			return true;
    }
}
```

HandlerInterceptor를 구현하여 IP 접근 제어용 interceptor를 생성한다.

Spring interceptor를 반드시 bean으로 등록해야하는건 아니지만 bean으로 등록하면 필요한 의존성을 편리하게 주입받을 수 있다.

추후에 로직에 필요한 bean을 주입받을 것이기 때문에 미리 @Component를 붙여두었다.

```java
@Configuration
@RequiredArgsConstructor
public class WebMvcConfig implements WebMvcConfigurer {

    private final IpAddressAccessControlInterceptor ipAddressAccessControlInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        registry.addInterceptor(ipAddressAccessControlInterceptor)
                .order(1)
                .addPathPatterns("/**");
    }
}
```

Interceptor는 WebMvcConfigurer 구현체에 addInterceptors를 override하여 등록한다.

addPathPatterns()에 접근 제어를 적용할 리소스 url를 설정한다.

`.addPathPatterns("/api/getData")` 와 같이 사용하면 되나 현재는 모든 url을 확인 하는 것으로 체크 하였다.

## Client Ip

IP 접근제어를 하려면 우선 클라이언트의 IP 주소를 가져와야한다.

이를 위한 유틸 메소드를 생성

```
public static String getClientIp(final HttpServletRequest request) {

    String ip = StringUtils.trimToNull(TextUtils.getSplitValue(request.getHeader("X-Forwarded-For"), ",", 0));

if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("x-real-ip");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("x-original-forwarded-for");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("Proxy-Client-IP");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("WL-Proxy-Client-IP");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("HTTP_X_FORWARDED_FOR");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("HTTP_X_FORWARDED");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("HTTP_X_CLUSTER_CLIENT_IP");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("HTTP_CLIENT_IP");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("HTTP_FORWARDED_FOR");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("HTTP_FORWARDED");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("HTTP_VIA");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getHeader("REMOTE_ADDR");
    }
if (ip == null || ip.length() == 0 || ip.equalsIgnoreCase("unknown")) {
        ip = request.getRemoteAddr();
    }

return ip;
}
```

다양한 종류의 proxy를 고려하여 가능한 header를 전부 확인하는 것이다.

이렇게 하지 않으면 확인이 되지않는 경우가 생기기도 한다.

이 메소드를 사용해 interceptor에서 클라이언트 IP를 가져올 수 있도록 코드를 추가한다.

```java
@RequiredArgsConstructor
@Component
public class IpAddressAccessControlInterceptor implements HandlerInterceptor {

    @Override
public booleanpreHandle(HttpServletRequest request, HttpServletResponse response, Object handler)throws Exception {
        final String clientIpAddress = WebUtils.getClientIp(request);
				return true;
    }
}
```

이후에는 2가지 방법이 있다.

## 1) IpAddressMatcher를 이용한 코드에 직접 등록

## IpAddressMatcher

IpAddressMatcher는 객체 생성 시에 접근을 허용할 IP 또는 IP 대역을 넘기도록 되어있다.

그리고 IP 체크는 matches() 메소드에 체크 대상 IP를 넘겨 결과를 받는다.

```
// IpAddressMatcher 객체 생성
IpAddressMatcher matcher =newIpAddressMatcher("192.168.1.0/24");

// match 여부 확인boolean result1 = matcher.matches("192.168.1.14");// trueboolean result2 = matcher.matches("127.0.0.1");// false
```

다음과 같이 허용할 IP 대역을 list로 별도의 클래스에서 관리할 수 있도록 manager를 만들었다.

```java
@Component
public class IpAddressMatcherManager {

    @Getter
		private List<IpAddressMatcher> ipAddressMatchers;

		public IpAddressMatcherManager() {

			this.ipAddressMatchers = List.of(
					newIpAddressMatcher("192.168.1.0/24"),
					newIpAddressMatcher("192.168.2.0/24"),
					newIpAddressMatcher("192.168.3.0/24")
        );
    }
}
```

다음으로 이 IpAddressMatcherManager를 사용하는 service를 만든다.

```
@RequiredArgsConstructor
@Service
publicclassIpAddressAccessControlService {

privatefinal IpAddressMatcherManager ipAddressMatcherManager;

public booleanisAccessible(String ipAddress) {
        List<IpAddressMatcher> ipAddressMatchers = ipAddressMatcherManager.getIpAddressMatchers();
return ipAddressMatchers.stream().anyMatch(matcher -> matcher.matches(ipAddress));
    }
}
```

isAccessible()은 파라미터로 받은 IP 주소가 접근 가능한지 여부를 반환하는 메소드이다.

IpAddressMatcherManager를 통해 가져온 matcher list에서 java stream의 anyMatch()를 사용해 간단하게 match 여부를 확인하는 로직을 구현할 수 있다.

이제 interceptor에서 이 service를 사용하여 접근 제어 로직을 완성해보자.

코드는 다음과 같다.

```java
@RequiredArgsConstructor
@Component
public class IpAddressAccessControlInterceptor implements HandlerInterceptor {

private final IpAddressAccessControlService ipAddressAccessControlService;

    @Override
public booleanpreHandle(final HttpServletRequest request, final HttpServletResponse response, Object handler)throws Exception {

        final String clientIpAddress = WebUtils.getClientIp(request);

if (!ipAddressAccessControlService.isAccessible(clientIpAddress)) {
            final String requestURI = request.getRequestURI();
            response.sendError(HttpServletResponse.SC_FORBIDDEN);
return false;
        }

return true;
    }
}
```

## 2) DB에서 가져오기

Domain 생성

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@Table(name = "white_list", catalog = "common")
@Builder
@Getter
public class WhiteList {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String ipAddress;
}
```

Repository 생성

```java
public interface WhiteListRepository extends JpaRepository<WhiteList, Long> {
}
```

확인 로직 적용

```java
@RequiredArgsConstructor
@Component
public class IpAddressAccessControlInterceptor implements HandlerInterceptor {

    private final WhiteListRepository whiteListRepository;

    @Override
    public boolean preHandle(final HttpServletRequest request, final HttpServletResponse response, Object handler) throws Exception {

        final String clientIpAddress = WebUtils.getClientIp(request);	// 추가
        if (this.whiteListRepository.findAll().stream().noneMatch(whiteList ->
                Objects.equals(whiteList.getIpAddress(), clientIpAddress))) {
            throw new Exception(clientIpAddress+ " Current Ip can not access!!");
        }

        return true;
    }
}
```
