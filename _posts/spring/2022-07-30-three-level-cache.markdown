---
layout: post
title:  "SpringBoot 循环依赖之三级缓存"
date:   2022-06-25 11:26:30 +0800
categories: Spring
---

# SpringBoot 循环依赖之三级缓存

## 1、循环依赖

## 2、三级缓存

### 2.1 Step1 获取一个单例

```java
    /**
     * 这里默认使用allowEarlyReference=true <br>
     * {@link org.springframework.beans.factory.support.AbstractBeanFactory#getSingleton(String)}
     *
     * @param beanName beanName
     * @return beanInstance
     */
    @Nullable
    public Object getSingleton(String beanName) {
        // 获取单例
        return getSingleton(beanName, true);
    }
```

### 2.2 Step1 获
