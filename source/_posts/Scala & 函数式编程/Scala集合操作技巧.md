---
layout: post
date: 2019-06-05 10:21
comments: true
toc: true
title: Scala集合操作技巧 
category: Scala & 函数式编程
tags: 
- Scala
---


# Scala集合操作技巧


原文: [Scala Collections Tips and Tricks](https://pavelfatin.com/scala-collections-tips-and-tricks/), 
作者[Pavel Fatin](https://pavelfatin.com/about/)是JetBrains 的一名员工，为神器IntelliJ IDEA开发Scala插件。 
受其工作[Scala Collections inspections](https://youtrack.jetbrains.com/issues/SCL?q=by%3A+Pavel.Fatin+collection+order+by%3A+created)的启发，他整理了这个关于Java Collections API技巧的列表。 
一些技巧只在一些微妙的实现细节中体现，但是大部分技巧都是一般的常识，但是在大部分情况下被忽视了。 
这些提示和技巧很有价值，可以帮你深入理解Scala Collections，可以是你的代码更快更简洁。 

## 1 图例 Legend

为了使下面的例子更容易理解，这里列出了一些约定：
  
约定 |  含义
---: | :-----
seq     |  一个Seq-based的集合, 比如Seq(1, 2, 3)
set     |  一个Set实例, 比如Set(1, 2, 3)
array   |  一个数组, 比如Array(1, 2, 3)
option  |  一个Option, 比如Some(1)
map     |  一个Map, 比如Map(1 -> "foo", 2 -> "bar")
p       |  一个断言predicate函数，类型T => Boolean, 比如_ > 2
n       |  一个整数
i       |  一个索引
f, g    |  简单函数,A => B
x, y    |  一些字面值(arbitrary values)
z       |  初始值或者缺省值


## 2 组合Composition
记住，尽管这些技巧都是独立和自包含的，我们还是可以将它们组合起来逐步迭代得到一个更高级的表达式，比如下面的例子，在一个Seq中检查一个元素是否存在：
```scala
seq.filter(_ == x).headOption != None

// Via seq.filter(p).headOption -> seq.find(p)
seq.find(_ == x) != None

// Via option != None -> option.isDefined
seq.find(_ == x).isDefined

// Via seq.find(p).isDefined -> seq.exists(p)
seq.exists(_ == x)

// Via seq.exists(_ == x) -> seq.contains(x)
seq.contains(x)
```
## 3 副作用 Side effects
”Side effects“是一个基础概念，在函数式编程语言中会有这个概念。 Scala有一个PPT专门介绍:[Side-effect checking for Scala](https://wiki.scala-lang.org/download/attachments/1310722/effects.pdf) 

基本上，”Side effects“是这样一个动作， 除了返回一个值外，外部函数或者表达式还观察到此动作还有以下行为之一:
+ 有输入输出操作 (比如文件，网络I/O)
+ 对外部变量有修改
+ 外部对象的状态有改变
+ 抛出异常

当一个函数或者表达式有以上任何一种情况时，我们就说它有副作用(Side effects),否则我们就说它是"纯"的函数或者表达式 (pure)。
side effects有什么大不了的？当有副作用时，计算的顺序不能随便改变。 比如下面两个"纯" (pure)表达式:
```scala
val x = 1 + 2
val y = 2 + 3
```
因为它们没有副作用，两个表达式可以互换位置，先x后y和先y后x的效果一样。 
如果有副作用 (有控制台输出)：
```scala
val x = { print("foo"); 1 + 2 }
val y = { print("bar"); 2 + 3 }
```
两个表达式不能互换位置，因为一旦互换位置，输出结果的顺序变了。 
所以副作用的一个影响就是会减少可能的转换的数量(reduces the number of possible transformations)，包括可能的简化和优化。 
同样的原因可适用用Collection相关的表达式上。看一个外部的builder变量（副作用方法append）:
```scala
seq filter { x => builder.append(x); x > 3 } headOption
```
原则上```seq.filter(p).headOption```可以简化为```seq.find(p)```，但是副作用阻止我们这么做. 如果你尝试这么做:
```scala
seq find { x => builder.append(x); x > 3 }
```
结果和前一个表达式并不一样。 前一个表达式计算后所有大于3的元素都增加到builder中了， 后一个表达式在找到第一个大于3的元素后就不会再增加了。 
两个表达式并不等价。

自动简化是否可能？这里有两条黄金法则，可以用在有副作用的代码上：
+ 1.尽可能的避免副作用    
+ 2.否则将副作用嗲吗从纯代码中分离开

对于上面的例子，我们需要去掉builder或者将它从纯代码中隔离。 考虑到builder是第三方的对象，我们无法去除，那我们通过隔离的方式实现：
```scala
seq.foreach(builder.append)seq.filter(_ > 3).headOption
```
这样我们就可以用到本文中的技巧进行替换：
```scala
seq.foreach(builder.append)seq.find(x > 3)
```
干的漂亮！自动简化也成为可能，一个额外好处是由于清晰的隔离，代码更容易理解。 
一个不太明显的好处是，代码变得更健壮。如上面的例子，副作用针对不同的Seq实现，副作用的结果也不相同， 比如Vector和Stream， 副作用隔离可以让我们避免这种不确定的行为。
## 4 Sequence

本节中的技巧针对Seq以及它的子类， 而一些转换可以应用于其它集合类，如Set,Option,Map,甚至Iterator类，因为它们提供了相近的接口。
### 4.1 创建Creation

显示创建集合
```scala
// Before
Seq[T]()

// After
Seq.empty[T]
```
有时候可以节省内存（重用empty对象）和CPU (length check浪费)。
也可应用于Set,Option,Map,Iterator.
### 4.2 Length

#### 4.2.1 对于数组，优先使用length而不是size

```scala
// Befor
earray.size

// After
array.length
```
length和size基本是同义词。在Scala 2.11中，Array.size是通过隐式转换实现的。因此每次调用时，一个中间包装类会被创建，除非你允许jvm的[escape analysis](https://en.wikipedia.org/wiki/Escape_analysis)。这会产生多余的GC对象，影响性能。
#### 4.2.2 不要对检查empty的属性取反

```scala
// Before
!seq.isEmpty
!seq.nonEmpty

// After
seq.nonEmpty
seq.isEmpty
```
同样适用于Set,Option,Map,Iterator
#### 4.2.3 不要通过计算length来检查empty

```scala
// Before
seq.length > 0
seq.length != 0
seq.length == 0

// After
seq.nonEmpty
seq.nonEmpty
seq.isEmpty
```
一方面已经有检查empty的方法，另一方面，比如LinearSeq和子类List,会花费O(n)的时间计算length(IndexedSeq花费O(1))。
同样适用于Set,Map
#### 4.2.4 不要直接使用length来比较

```scala
// Before
seq.length > n
seq.length < n
seq.length == n
seq.length != n

// After
seq.lengthCompare(n) > 0
seq.lengthCompare(n) < 0
seq.lengthCompare(n) == 0
seq.lengthCompare(n) != 0
```
同上一条，计算length有时候非常昂贵,有可能将花费从O(length)减少到O(length min n)。 
对于无限的stream来说，上面的技巧是绝对必要的。
### 4.3 等价 Equality

#### 4.3.1 不要使用==比较数组

```
// Before
array1 == array2

// After
array1.sameElements(array2)
```
因为==只是比较实例对象，而不是里面的元素。
同样适用于Iterator
#### 4.3.2 不要检查不同分类(categories)的集合的相等性

```scala
// Before
seq == set

// After
seq.toSet == set
```
#### 4.3.3 不要使用sameElements来比较有序集合

```scala
// Before
seq1.sameElements(seq2)

// After
seq1 == seq2
```
#### 4.3.4 不要手工检查相等性

```scala
// Before
seq1.corresponds(seq2)(_ == _)

// After
seq1 == seq2
```
使用**_内建_**的方法。
### 4.4 Indexing

#### 4.4.1 不要使用index得到第一个元素

```scala
// Before
seq(0)

// After
seq.head
```
#### 4.4.2 不要使用index得到最后一个元素

```scala
// Before
seq(seq.length - 1)

// After
seq.last
```
#### 4.4.3 不要显式检查index的边界

```scala
// Before
if (i < seq.length) Some(seq(i)) else None

// After
seq.lift(i)
```
#### 4.4.4 不要仿造headOption

```scala
// Before
if (seq.nonEmpty) Some(seq.head) else None
seq.lift(0)

// After
seq.headOption
```
#### 4.4.5 不要仿造lastOption

```scala
// Before
if (seq.nonEmpty) Some(seq.last) else None
seq.lift(seq.length - 1)

// After
seq.lastOption
```
#### 4.4.6 小心indexOf和lastIndexOf参数类型

```scala
// Before
Seq(1, 2, 3).indexOf("1") // compilable
Seq(1, 2, 3).lastIndexOf("2") // compilable

//  After
Seq(1, 2, 3).indexOf(1)
Seq(1, 2, 3).lastIndexOf(2)
```
#### 4.4.7 不要构造index的Range

```scala
// Before
Range(0, seq.length)

// After
seq.indices
```
#### 4.4.8 不要手工使用index来zip集合

```scala
// Before
seq.zip(seq.indices)

// After
seq.zipWithIndex
```
### 4.5 检查元素的存在 Existence

#### 4.5.1 不要用断言equality predicate 来检查存在

```scala
// Before
seq.exists(_ == x)

//  After
seq.contains(x)
```
同样应用于Set,Option,Iterator
#### 4.5.2 小心contains参数类型

```scala
// Before
Seq(1, 2, 3).contains("1") // compilable

//  After
Seq(1, 2, 3).contains(1)
```
#### 4.5.3 不要用断言inequality predicate 来检查不存在

```scala
// Before
seq.forall(_ != x)

// After
!seq.contains(x)
```
同样应用于Set,Option,Iterator
#### 4.5.4 不要统计元素的数量来检查存在

```scala
// Before
seq.count(p) > 0
seq.count(p) != 0
seq.count(p) == 0

//  After
seq.exists(p)
seq.exists(p)
!seq.exists(p)
```
同样应用于Set,Map,Iterator
#### 4.5.5 不要借助filter来检查存在

```scala
// Before
seq.filter(p).nonEmpty
seq.filter(p).isEmpty

// After
seq.exists(p)
!seq.exists(p)
```
同样应用于Set,Option,Map,Iterator
### 4.6 Filtering

#### 4.6.1 不要对断言取反

```scala
// Before
seq.filter(!p)

// After
seq.filterNot(p)
```
同样应用于Set,Option,Map,Iterator
#### 4.6.2 不要借助filter统计元素数量

```scala
// Before
seq.filter(p).length

// After
seq.count(p)
```
调用filter会产生一个临时集合，影响GC和性能。
同样应用于Set,Option,Map,Iterator
#### 4.6.3 不要借助filter找到元素的第一个值

```scala
// Before
seq.filter(p).headOption

// After
seq.find(p)
```
同样应用于Set,Option,Map,Iterator
### 4.7 Sorting

#### 4.7.1 不要手工按一个属性排序

```scala
// Before
seq.sortWith(_.property <  _.property)

// After
seq.sortBy(_.property)
```
#### 4.7.2 不要手工按照identity排序

```scala
// Before
seq.sortBy(it => it)
seq.sortBy(identity)
seq.sortWith(_ < _)

// After
seq.sorted
```
#### 4.7.3 一步完成排序反转

```scala
// Before
seq.sorted.reverse
seq.sortBy(_.property).reverse
seq.sortWith(f(_, _)).reverse

// After
seq.sorted(Ordering[T].reverse)
seq.sortBy(_.property)(Ordering[T].reverse)
seq.sortWith(!f(_, _))
```
### 4.8 Reduction

#### 4.8.1 不要手工计算sum

```scala
// Before
seq.reduce(_ + _)
seq.fold(z)(_ + _)

// After
seq.sum
seq.sum + z
```
其它可能用的方法reduceLeft,reduceRight,foldLeft,foldRight
同样应用于Set,Iterator
#### 4.8.2 不要手工计算product

```scala
// Before
seq.reduce(_ * _)
seq.fold(z)(_ * _)

// After
seq.product
seq.product * z
```
同样应用于Set,Iterator
#### 4.8.3 不要手工搜索最小值和最大值

```scala
// Before
seq.reduce(_ min _)
seq.fold(z)(_ min _)

// After
seq.min
z min seq.min
```
```scala
// Before
seq.reduce(_ max _)
seq.fold(z)(_ max _)

// After
seq.max
z max seq.max
```
同样应用于Set,Iterator
#### 4.8.4 不要仿造forall

```scala
// Before
seq.foldLeft(true)((x, y) => x && p(y))
!seq.map(p).contains(false)

// After
seq.forall(p)
```
同样应用于Set,Option(for the second line),Iterator
#### 4.8.5 不要仿造exists

```scala
// Before
seq.foldLeft(false)((x, y) => x || p(y))
seq.map(p).contains(true)

// After
seq.exists(p)
```
### 4.9 Rewriting

#### 4.9.1 合并连续的filter调用

```scala
// Before
seq.filter(p1).filter(p2)

// After
seq.filter(x => p1(x) && p2(x))
```
或`seq.view.filter(p1).filter(p2).force`
同样应用于Set,Option,Map,Iterator
#### 4.9.2 合并连续的map调用

```scala
// Before
seq.map(f).map(g)

// After
seq.map(f.andThen(g))
```
或`seq.view.map(f).map(g).force`
同样应用于Set,Option,Map,Iterator
#### 4.9.3 filter完后再排序

```scala
// Before
seq.sorted.filter(p)

// After
seq.filter(p).sorted
```
#### 4.9.4 在调用map前不要显式调用反转reverse

```scala
// Before
seq.reverse.map(f)

// After
seq.reverseMap(f)
```
#### 4.9.5 不要显示反转得到迭代器

```scala
// Before
seq.reverse.iterator

// After
seq.reverseIterator
```
#### 4.9.6 不要通过将集合转成Set得到不重复集合

```scala
// Before
seq.toSet.toSeq

// After
seq.distinct
```
#### 4.9.7 不要仿造slice

```scala
// Before
seq.drop(x).take(y)

// After
seq.slice(x, x + y)
```
同样应用于Set,Map,Iterator
#### 4.9.8 不要仿造splitAt

```scala
// Before
val seq1 = seq.take(n)
val seq2 = seq.drop(n)

// After
val (seq1, seq2) = seq.spiltAt(n)
```
#### 4.9.9 不要仿造span

```scala
// Before
val seq1 = seq.takeWhile(p)
val seq2 = seq.dropWhile(p)

// After
val (seq1, seq2) = seq.span(p)
```
#### 4.9.10 不要仿造partition

```scala
// Before
val seq1 = seq.filter(p)
val seq2 = seq.filterNot(p)

// After
val (seq1, seq2) = seq.partition(p)
```
#### 4.9.11 不要仿造takeRight

```scala
// Before
seq.reverse.take(n).reverse

// After
seq.takeRight(n)
```
#### 4.9.12 不要仿造flatten

```scala
// Before (seq: Seq[Seq[T]])
seq.flatMap(it => it)
seq.flatMap(identity)

// After
seq.flatten
```
同样应用于Set,Map,Iterator
#### 4.9.13 不要仿造flatMap

```scala
// Before (f: A => Seq[B])
seq.map(f).flatten

// After
seq.flatMap(f)
```
同样应用于Set,Option,Iterator
#### 4.9.14 不需要结果时不要用map

```scala
// Before
seq.map(...) // the result is ignored

// After
seq.foreach(...)
```
同样应用于Set,Option,Map,Iterator
#### 4.9.15 不要产生临时集合

1.使用view
```scala
// Before
seq.map(f).flatMap(g).filter(p).reduce(...)

// After
seq.view.map(f).flatMap(g).filter(p).reduce(...)
```

2.将view转换成一个同样类型的集合
```scala
// Before
seq.map(f).flatMap(g).filter(p)
// After
seq.view.map(f).flatMap(g).filter(p).force
```
如果中间的转换是filter，还可以
```scala
seq.withFilter(p).map(f)
```

3.将view转换成另一种集合
```scala
// Before
seq.map(f).flatMap(g).filter(p).toList

// After
seq.view.map(f).flatMap(g).filter(p).toList
```
还有一种“transformation + conversion”方法：
```scala
seq.map(f)(collection.breakOut): List[T]
```
#### 4.9.16 使用赋值操作符

```scala
// Before
seq = seq :+ x
seq = x +: seq
seq1 = seq1 ++ seq2
seq1 = seq2 ++ seq1

// After
seq :+= x
seq +:= x
seq1 ++= seq2
seq1 ++:= seq2
```
Scala有一个语法糖，自动将x <op>= y转换成x = x <op> y. 如果op以:结尾，则被认为是右结合的操作符。 
一些list和stream的语法：
```scala
// Before
list = x :: list
list1 = list2 ::: list
stream = x #:: list
stream1 = stream2 #::: stream

// After
list ::= x
list1 :::= list2
stream #::= x
stream1 #:::= stream2
```
同样应用于Set,Map,Iterator
## 5 Set

大部分的Seq的技巧也可以应用于Set。另外还有一些只针对Set的技巧。
### 5.1 不要使用sameElements比较未排序的集合

```scala
// Before
set1.sameElements(set2)

// After
set1 == set2
```
同样应用于Map
### 5.2 不要手工计算交集

```scala
// Before
set1.filter(set2.contains)
set1.filter(set2)

// After
set1.intersect(set2) 
// or 
set1 & set2
```
### 5.3 不要手工计算diff

```scala
// Before
set1.filterNot(set2.contains)
set1.filterNot(set2)

// After
set1.diff(set2) 
// or 
set1 &~ set2
```
## 6 Option

Option并不是集合类，但是它提供了类似的方法和行为。 
大部分针对Seq的技巧也适用于Option。这里列出了一些特殊的只针对Option的技巧。
### 6.1 Value

#### 6.1.1 不要使用None和Option比较

```scala
// Before
option == None
option != None

// After
option.isEmpty
option.isDefined
```
#### 6.1.2 不要使用Some和Option比较

```scala
// Before
option == Some(v)
option != Some(v)

// After
option.contains(v)
!option.contains(v)
```
#### 6.1.3 不要使用实例类型来检查值的存在性

```scala
// Before
option.isInstanceOf[Some[_]]

// After
option.isDefined
```
#### 6.1.4 不要使用模式匹配来检查值的存在

```scala
// Before
option match {    
    case Some(_) => true    
    case None => false
}

// After
option.isDefined
```
同样适用于Seq,Set
#### 6.1.5 对于检查存在性的属性不要取反

```scala
// Before
!option.isEmpty
!option.isDefined
!option.nonEmpty

// After
seq.isDefined
seq.isEmpty
seq.isEmpty
```
#### 6.1.6 不要检查值的存在性再处理值

```scala
// Before
if (option.isDefined) {    
    val v = option.get    
    ...
}

// After
option.foreach { v =>    
    ...
}
```
### 6.2 Null

#### 6.2.1 不要通过和null比较来构造Option

```scala
// Before
if (v != null) Some(v) else None

// After
Option(v)
```
#### 6.2.2 不要显示提供null作为备选值

```scala
// Before
option.getOrElse(null)

// After
option.orNull
```
### 6.3 Rewriting

#### 6.3.1 将mapwithgetOrElse转换成fold

```scala
// Before
option.map(f).getOrElse(z)

// After
option.fold(z)(f)
```
#### 6.3.2 不要仿造exists

```scala
// Before
option.map(p).getOrElse(false)

// After
option.exists(p)
```
#### 6.3.3 不要手工将option转换成sequence

```scala
// Before
option.map(Seq(_)).getOrElse(Seq.empty)
option.getOrElse(Seq.empty) // option: Option[Seq[T]]

// After
option.toSeq
```
## 7 Map

同上，这里只列出针对map的技巧
### 7.1 不要使用lift替换get

```scala
// Before
map.lift(n)

// After
map.get(n)
```
因为没有特别的需要将map值转换成一个Option。
### 7.2 不要分别调用get和getOrElse

```scala
// Before
map.get(k).getOrElse(z)

// After
map.getOrElse(k, z)
```
### 7.3 不要手工抽取键集合

```scala
// Before
map.map(_._1)
map.map(_._1).toSet
map.map(_._1).toIterator

// After
map.keys
map.keySet
map.keysIterator
```
### 7.4 不要手工抽取值集合

```scala
// Before
map.map(_._2)
map.map(_._2).toIterator

// After
map.values
map.valuesIterator
```
### 7.5 小心使用filterKeys

```scala
// Before
map.filterKeys(p)

// After
map.filter(p(_._1))
```
因为filterKeys包装了原始的集合，并没有复制元素，后续处理得小心。
### 7.6 小心使用mapValues

```scala
// Before
map.mapValues(f)

// After
map.map(f(_._2))
```
同上。
### 7.7 不要手工filter 键

```scala
// Before
map.filterKeys(!seq.contains(_))

// After
map -- seq
```
### 7.8 使用赋值操作符重新赋值

```scala
// Before
map = map + x -> y
map1 = map1 ++ map2
map = map - x
map = map -- seq

// After
map += x -> y
map1 ++= map2
map -= x
map --= seq
```
## 8 补充

除了以上的介绍，建议你看一下官方文档[Scala Collections documentation](http://docs.scala-lang.org/overviews/collections/introduction.html)。
还有
+ [Scala for Project Euler](https://pavelfatin.com/scala-for-project-euler/)
+ [Ninety-nine](https://pavelfatin.com/ninety-nine/)

原文链接: [Scala Collections 提示和技巧](http://colobu.com/2015/07/02/Scala-Collections-Tips-and-Tricks/)

