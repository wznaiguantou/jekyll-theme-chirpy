---
layout: post
title:  "SpringBoot 日志框架"
date:   2022-05-02 11:26:30 +0800
categories: Spring
---

# SpringBoot 日志框架



## 1. 日志框架启动的选择

```java
    private static final LogAdapter.LogApi logApi;

    static {
        if (isPresent(LOG4J_SPI)) {
            if (isPresent(LOG4J_SLF4J_PROVIDER) && isPresent(SLF4J_SPI)) {
                // log4j-to-slf4j bridge -> we'll rather go with the SLF4J SPI;
                // however, we still prefer Log4j over the plain SLF4J API since
                // the latter does not have location awareness support.
                logApi = LogAdapter.LogApi.SLF4J_LAL;
            } else {
                // Use Log4j 2.x directly, including location awareness support
                logApi = LogAdapter.LogApi.LOG4J;
            }
        } else if (isPresent(SLF4J_SPI)) {
            // Full SLF4J SPI including location awareness support
            logApi = LogAdapter.LogApi.SLF4J_LAL;
        } else if (isPresent(SLF4J_API)) {
            // Minimal SLF4J API without location awareness support
            logApi = LogAdapter.LogApi.SLF4J;
        } else {
            // java.util.logging as default
            logApi = LogAdapter.LogApi.JUL;
        }
    }

    private static boolean isPresent(String className) {
        try {
            Class.forName(className, false, LogAdapter.class.getClassLoader());
            return true;
        } catch (ClassNotFoundException ex) {
            return false;
        }
    }
```





