---
layout: post
date: 2019-06-05 10:21
comments: true
toc: true
title: Scala常用语法技巧 
category: Scala & 函数式编程
tags: 
- Scala
---


# Scala常用语法技巧

## 1. 多参数列表

Scala 允许你指明函数的**最后一个参数**可以是重复的。这可以允许客户向函数传入可变长度参数列表。想要标注一个重复参数，在参数的类型之后加一个 `*` 。例如：

```scala
def echo(args: String*) = {
  for (arg <- args) println(arg)
}
```

这样定义， echo 可以被零个至多个 String 参数调用

```scala
echo()
echo("one")
echo("hello", "world")
```

函数内部，重复参数的类型是声明参数类型的数组。因此， echo 函数里被声明为类型“ String* ”
的 args 的类型实际上是 Array[String] 。然而，如果你有一个合适类型的数组，并尝试把它当作
重复参数传入，你会得到一个编译器错误：

```scala
val arr = Array("I`m", "from", "China")
echo(arr)

# 编译报错
error: type mismatch;

found : Array[java.lang.String]
required: String
```

要实现这个做法，你需要在数组参数后添加 `: _*` 符号，像这样：

```scala
val arr = Array("I`m", "from", "China")
echo(arr: _*)
```

这个标注 `: _*` 告诉编译器把 arr 的每个元素当作参数，而不是当作单一的参数传给 echo 。

## 2. Tuple2与Map
```scala
Map(1->"a", 2->"b").map { case (i, j)=> 
  (1, j)
}

res0: scala.collection.immutable.Map[Int,String] = Map(1 -> b)
```
如果map中返回的是Tuple2类型, 那么最后结果会转化成Map, 同时相同的key会被覆盖掉

```scala
Map(1->"a", 2->"b").map { case (i, j)=> 
  (1, 1, j)
}

res3: scala.collection.immutable.Iterable[(Int, Int, String)] = List((1,1,a), (1,1,b))
```
如果map中返回的不是Tuple2类型, 那么最后结果就是List而不是Map, 此时就不必担心数据被覆盖的问题了