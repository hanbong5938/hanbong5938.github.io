---
layout : post
title : "Multi Module"
category : Gradle
---

## 구성
```
root - app
     - auth
```

루트에는 아래 와 같이 setting.gradle.kts 등의 파일이 필요하나
서브 프로젝트에는 build.gradle.kts 만 있으면 된다.

![스크린샷](https://user-images.githubusercontent.com/51283645/217818609-be8fb66c-6f46-4cae-82c0-b93b24293a91.png)


### root 의 build.gradle.kts
```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.0.2"
    id("io.spring.dependency-management") version "1.1.0"
}

java.sourceCompatibility = JavaVersion.VERSION_17


allprojects {
    group = "com.demo"
    version = "0.0.1-SNAPSHOT"


    repositories {
        mavenCentral()
    }
}

subprojects {
    apply(plugin = "java")
    apply(plugin = "org.springframework.boot")
    apply(plugin = "io.spring.dependency-management")

    dependencies {
        implementation("org.springframework.boot:spring-boot-starter")
        testImplementation("org.springframework.boot:spring-boot-starter-test")
    }

    tasks.withType<Test> {
        useJUnitPlatform()
    }
}

project(":app") {
}

project(":auth") {
}

```

### setting.gradle.kts
```kotlin
rootProject.name = "root"
include(
    "auth",
    "app"
)
```

### app 의 build.gradle.kts
```kotlin
dependencies {
}

```

### auth 의 build.gradle.kts
```kotlin
dependencies {
}
```

필요한 dependency 는 각 서브 프로젝트에서 추가하면 된다.