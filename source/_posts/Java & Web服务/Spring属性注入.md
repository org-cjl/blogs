---
layout: post
date: 2019-08-05 10:30
comments: true
toc: true
title: Spring属性注入
category: Java & Web服务
tags:
- Spring
---

# Spring 属性注入

## 1. `@Value`注入属性

```java
  @Value("normal")
  private String normal; // 注入普通字符串

  @Value("#{systemProperties['os.name']}")
  private String systemPropertiesName; // 注入操作系统属性

  @Value("#{ T(java.lang.Math).random() * 100.0 }")
  private double randomNumber; //注入表达式结果

  @Value("#{beanInject.another}")
  private String fromAnotherBean; // 注入其他Bean属性：注入beanInject对象的属性another，类具体定义见下面

  @Value("classpath:com/hry/spring/configinject/config.txt")
  private Resource resourceFile; // 注入文件资源

  @Value("http://www.baidu.com")
  private Resource testUrl; // 注入URL资源
```