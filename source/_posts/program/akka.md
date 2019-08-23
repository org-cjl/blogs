---
layout: post
date: 2019-06-05 10:21
comments: true
toc: true
title: akka 
tags: 
- Scala
---

# Akka

## 1. `PoisonPill`

所有Actor都能处理的消息, 处理时将永久终止Actor(自杀毒药)

```scala
class Pinger extends Actor {

  override def receive: Receive = {
    case Pong =>
      println(s"${self.path} received pong")
      // PoisonPill: 所有Actor都能处理的消息, 处理时将永久终止Actor
      sender() ! PoisonPill
      self ! PoisonPill
    case _ =>
  }
}
```

