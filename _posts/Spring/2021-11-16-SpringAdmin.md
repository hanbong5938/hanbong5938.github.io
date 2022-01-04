---
layout : post
title : "SpringBootAdmin"
category : Spring
---
# **spring-boot-admin-server**

## **어드민 서버**

### **그래들**

```bash
implementation("org.springframework.boot:spring-boot-starter-security:2.5.6")
// https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web
implementation("org.springframework.boot:spring-boot-starter-web:2.5.6")
// https://mvnrepository.com/artifact/de.codecentric/spring-boot-admin-starter-server
implementation("de.codecentric:spring-boot-admin-starter-server:2.5.4")
```

### **메인 메서드**

- 어노테이션 추가

```bash
@EnableAdminServer
```

### **시큐리티 설정**

```bash

@Configuration
class SecurityConfig : WebSecurityConfigurerAdapter() {

    var adminServerProperties = AdminServerProperties();

    override fun configure(http: HttpSecurity?) {
        val contextPath = this.adminServerProperties.contextPath

        val successHandler = SavedRequestAwareAuthenticationSuccessHandler()
        successHandler.setTargetUrlParameter("redirectTo")
        successHandler.setDefaultTargetUrl("/")

        http!!.authorizeRequests()
            .antMatchers("$contextPath/assets/**").permitAll()
            .antMatchers("$contextPath/login").permitAll()
            .antMatchers(HttpMethod.GET, "/actuator/**").permitAll()
            .anyRequest().authenticated()
            .and()
            .formLogin().loginPage("$contextPath/login").successHandler(successHandler)
            .and()
            .logout().logoutUrl("$contextPath/logout")
            .and()
            .httpBasic()
            .and()
            .csrf()
            .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            .ignoringAntMatchers(
                "$contextPath/instances",
                "$contextPath/actuator/**"
            )
    }
}
```

### yml

```bash
server:
  port: 50000
spring:
  security:
    user:
      name: admin
      password: admin
#boot:
  #  admin:
  #   ui:
  #     public-url: https://gorani.wecruitpro.com:443/  # 프록시 사용하는 경우
```

## **관리하는 프로젝트**

### **yml**

```bash
management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always
spring:
  boot:
    admin:
      client:
        auto-registration: 'true'
        password: admin
        url: http://localhost:50000 # admin url
        username: admin
```

### **그래들**

```bash
// https://mvnrepository.com/artifact/de.codecentric/spring-boot-admin-starter-client
implementation 'de.codecentric:spring-boot-admin-starter-client:2.5.4'
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

[SpringBootAdmin](https://github.com/hanbong5938/spring-boot-admin-server)
