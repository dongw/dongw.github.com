---
layout: post
title: "在Scala中如何避免导入implicit相关的定义"
date: 2011-03-11 13:06
comments: true
categories: [Scala, implicit]
---
Scala中的implicit能够让代码变的简洁很多。很多时候我们倾向于把这些implicit相关定义放到一个统一的地方，然后在各个package中应用。但每次用的时候，都需要做类是这样的import：

``` scala
    import com.readventure.MyImplicits._
```
下面我告诉大家一个简单的方式，可以避免这种没有必要的imports。

##方法

我们可以在com/readventure/下面定义下面这个trait：
``` scala
package com.readventure

trait MyImplicits {
  implicit def str2opt(s: String) = Option(s)
  ...
}
```
然后在每个要用到这些implicit的包下面（比如`com.readventure.test1`)放置这样叫`package.scala`的文件（文件名不重要）:
``` scala
package com.readventure  // not 'package.readventure.test1'
package object test1 extends a.Implicits { /* your other stuff goes here */}
```

注意，这里的package的名字，以及package object的名字很重要，必须和相对应的路径对应。这样在`MyImplicits`里面的所有东西在`com.readventure.test1`的任何一个类里面都可以用了，不用显式import任何东西。比如：
``` scala
package com.readventure.test1

class SomeClass {
  def testImplicit(str: String): Option[String] = str
}
```

`Package object`是Scala2.8的新特性，如果有兴趣，可以看看[这里](http://www.artima.com/scalazine/articles/package_objects.html)。
