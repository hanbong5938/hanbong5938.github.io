---
layout : post
title : "Request Header 언어 읽기"
category : Spring
---
```javascript
customAxios.interceptors.request.use(function (req) {
    req.headers["Accept-Language"] = 'en';
    return req;
});
```
axios 공통 설정을 통해 정보를 넣어준다.

```java
@Component
public class Message {

    private static MessageSource messageSource;

    public Message(MessageSource messageSource) {
        Message.messageSource = messageSource;
    }

    public static String getMessage(String str) {
        return messageSource.getMessage(str, null, LocaleContextHolder.getLocale());
    }

    public static String getMessage(String str, String[] arr) {
        return messageSource.getMessage(str, arr, LocaleContextHolder.getLocale());
    }

}
```
위와 같이 사용하면 처리 가능


만약 Request에서 정보가 없는 경우 디폴트로

```java
Locale.getDefault();
```
의 값을 가져온다.