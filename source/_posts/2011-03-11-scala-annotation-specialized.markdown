---
layout: post
title: "有关Scala的 @Specialized Annotation"
date: 2011-03-11 11:00
comments: true
categories: [Scala, specialized]
---


@specialized 主要用在Scala的范型上，使得Scala编译器能够在编译的时候，为某些primative类型提供一个更加高效的实现。这种高效源于避免了对指定primative类型的boxing和unboxing(int -> Int -> int)[^1]。根据一个相关研究[^2]，boxing和unboxing还是很花时间的。比如

``` scala
	class My [A] {
  	  def iden(x: A): A = x
	}
	val a = new My[Int]
	for (i <- 1 to 1000000) a.iden(i)
```

需要花40纳秒。改为：
``` scala
class My [@specialized(Int) A] {
    def iden(x: A): A = x
}
val a = new My[Int]
for (i <- 1 to 1000000) a.iden(i)
```
就只需要17纳秒。

下面看一个简单的用`@specialized`的例子：
``` scala
class Example[@specialized(Int) T](value: T) {
  def get(): T = value
}
```
用命令`scalac -print test.scala`编译后变为：
``` scala
        [[syntax trees at end of cleanup]]// Scala source: test.scala 
        package <empty> { 
          class Example extends java.lang.Object with ScalaObject { 
            <paramaccessor> protected[this] val value: java.lang.Object = _; 
            def get(): java.lang.Object = Example.this.value; 
            <specialized> def get$mcI$sp(): Int = scala.Int.unbox(Example.this.get()); 
            def this(value: java.lang.Object): Example = { 
              Example.this.value = value; 
              Example.super.this(); 
              () 
            } 
          }; 
          <specialized> class Example$mcI$sp extends Example { 
            <paramaccessor> <specialized> protected[this] val value$mcI$sp: Int = _; 
            override <specialized> def get(): Int = Example$mcI$sp.this.get$mcI$sp(); 
            override <specialized> def get$mcI$sp(): Int = Example$mcI$sp.this.value$mcI$sp; 
            override <bridge> <specialized> def get(): java.lang.Object = scala.Int.box(Example$mcI$sp.this.get()); 
            <specialized> def this(value$mcI$sp: Int): Example$mcI$sp = { 
              Example$mcI$sp.this.value$mcI$sp = value$mcI$sp; 
              Example$mcI$sp.super.this(scala.Int.box(value$mcI$sp)); 
              () 
            } 
          }
        }
```

可见有两个类被生成了。如果把源文件变为：
``` scala
class Example[@specialized(Int, Long) T](value: T) {
  def get(): T = value
}
```
则有第三个类被生成：   
``` scala
        [[syntax trees at end of cleanup]]// Scala source: test.scala
        package <empty> {
          ...
          ...
          <specialized> class Example$mcJ$sp extends Example {
            <paramaccessor> <specialized> protected[this] val value$mcJ$sp: Long = _;
            override <specialized> def get(): Long = Example$mcJ$sp.this.get$mcJ$sp();
            override <specialized> def get$mcJ$sp(): Long = Example$mcJ$sp.this.value$mcJ$sp;
            override <bridge> <specialized> def get(): java.lang.Object = scala.Long.box(Example$mcJ$sp.this.get());
            <specialized> def this(value$mcJ$sp: Long): Example$mcJ$sp = {
              Example$mcJ$sp.this.value$mcJ$sp = value$mcJ$sp;
              Example$mcJ$sp.super.this(scala.Long.box(value$mcJ$sp));
              ()
            }
          }
        }
```    	
理想情况是很多范型都对所有primative类型做specialized，不过这样编译后的代码就会成倍的增加了。Scala 2.8.1中只有下面的类有用到@specialized[^3]:
`Function0, Function1, Function2, Tuple1, Tuple2, Product1, Product2, AbstractFunction0, AbstractFunction1, AbstractFunction2`.