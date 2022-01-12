---
layout : post
title : "Controller Interface 이용하기"
category : Spring
---

아래와 같이 설정해준 다음

```java

@RequestMapping("/default")
public interface Test {

    @GetMapping("/")
    List<TestDto> getAll();

    @GetMapping("/{id}")
    Optional<TestDto> getById(@PathVariable int id);

    @PostMapping("/save/{id}")
    void save(@RequestBody TestDto testDto, @PathVariable int id);
}

```


```java
@RestController
@RequestMapping("/test")
@RequiredArgsConstructor
public class TestController implements Test {

    private final TestService testService;
    
    @Override
    public List<TestDto> getAll() {
        this.testService.getAll();
    }

    @Override
    public Optional<TestDto> getById(int id) {
        this.testService.getById(id);
    }

    @Override
    public void save(TestDto testDto, int id) {
        this.testService.save(testDto, id);
    }

}
```

테스트 전송
```http request
# curl http://localhost:8080/test/
GET http://localhost:8080/test/

###
```
