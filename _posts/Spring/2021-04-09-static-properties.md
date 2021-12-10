---
layout : post
title : "static properties"
category : Spring
---
```java

    public static String SERVICE_URL;
    public static String FILE_URL;

    @Value("${temp.service-url}")
    public void setServiceUrl(String value) {
        SERVICE_URL = value;
    }

    @Value("${temp.file-url}")
    public void setFileUrl(String value) {
        FILE_URL = value;
    }
```
