---
layout : post
title : "response null 인 경우 생략"
category : Spring
---
Bean이 아닌 클래스에서 Bean을 주입이 필요한 경우 사용한다.

static 메서드나 new 를 사용하여 생성한 인스턴스에서 Bean을 해야하는 경우가 그 예다.

### `ApplicationContextAware` 

`ApplicationContextAware` 인터페이스를 구현 한다.

applicationContext 를 이용해 bean을 가져올 클래스를 정의하고 사용하면 된다.

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class ApplicationContextProvider implements ApplicationContextAware {

    private static ApplicationContext context;

    @Override
    public void setApplicationContext(final ApplicationContext applicationContext) {
        context = applicationContext;
    }

    public static ApplicationContext getApplicationContext() {
        return context;
    }
}
```

```java
@UtilityClass
public class ApplicationContextUtils {
    public static <T> @NotNull T getBean(final Class<T> type) {
        final ApplicationContext applicationContext = ApplicationContextProvider.getApplicationContext();
        return applicationContext.getBean(type);
    }
}
```

### 사용 예제

```java
@Service
public class TestService {
    
    public void test() {
        System.out.println("Hello");
    }

}
```

```java
public class StaticTest {
    
    public static check() {
        final Test test = ApplicationContextUtils.getBean(TestService.class).test(properties);
    }
}
```
