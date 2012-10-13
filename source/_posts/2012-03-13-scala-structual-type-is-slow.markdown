---
layout: post
title: "Scala的structual type的确慢不少"
date: 2012-03-13 01:12
comments: true
categories: [Scala, Strutrual type, Performance]
---

Scala的Structual Type是用反射机制实现的。所以会慢一些。但我关心的是我经常用到的一个情况会慢多少。比如：
``` scala
type HasId = {
  def id: Long
}
case class Foo(val id: Long)
```
我经常会这样用：
``` scala
def loop1(foos: Seq[HasId]) = {
  var i = 0L
  foos.foreach(f => i += f.id)
  println(i)
}
```
这样所有有id的方法我都可以用这个`loop1`方法了。如果不用Structual Type，可以这样做：
``` scala
def loop2(foos: Seq[Foo]) = {
  var i = 0L
  foos.foreach(f => i += f.id)
  println(i)
}
```
在我的工作站上，如果给定一个10000000这么大的Seq[Foo]，`loop1`用了550ms,`loop2`用了262ms。

##结论
在这种情况下，用Structual Type的效率降低一半左右。