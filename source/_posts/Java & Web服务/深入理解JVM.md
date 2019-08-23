---
layout: post
date: 2019-08-05 10:30
comments: true
toc: true
title: 深入理解JVM
category: Java & Web服务
tags:
- JVM
---

# 深入理解JVM

> 参考资源:
- [x] [Java 学习/面试指南](https://snailclimb.gitee.io/javaguide/#/)
- [x] [深入理解JAVA虚拟机: JVM高级特性与最佳实践](https://github.com/doocs/jvm)

## 1. 线程的6种状态

状态名称      | 说明
:---------- |:---------
NEW          | 初始状态, 线程被构建, 但是还没有调用start方法
RUNNABLE     | 运行状态, Java线程将操作系统中的就绪和运行两种状态笼统的称作"运行中"
BLOCKED      | 阻塞状态, 表示线程阻塞于锁
WAITING      | 等待状态, 表示当前线程需要等待其他线程做出一些特定动作(通知或中断)
TIME_WAITING | 超时等待状态, 该状态不同于WAITING, 它可以在指定的时间自行返回
TERMINATED   | 终止状态, 表示当前线程已经执行完毕


> Java线程状态变迁

![Java 线程状态变迁](https://user-images.githubusercontent.com/17871962/62363910-d4952280-b552-11e9-9285-c77d3b1d214d.png)
