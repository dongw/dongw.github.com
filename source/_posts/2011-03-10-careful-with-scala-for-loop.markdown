---
layout: post
title: "Scala的循环可能很慢！"
date: 2011-03-10 10:39
comments: true
categories: [Scala, performance]
---


下面这Java段代码只要90纳秒[^1]:

``` scala
int s = 0
for (int i = 1; i <= 2000; i++)
  for (int j = 1; j <= 2000; j++)
    for (int k = 1; k <= 2000; k++)
      s += 1
```
而相对应的Scala则是需要1.3秒

``` scala
var s = 0
for (i <- 1 to 2000; 
     j <- 1 to 2000; 
     k <- 1 to 2000) 
  s += 1
```
同样，下面的代码和上面的完全等价：

``` scala
(1 to 2000).foreach(
  i => (1 to 2000).foreach(
    j => (1 to 2000).foreach(k => s += 1)
  )
)
```

改为简单Scala for循环就可以快到60纳秒：

``` scala
	var s = 0
	var i = 1; var j = 1; var k = 1

    while (i <= 2000) { j = 1
      while (j <= 2000) { k = 1
        while (k <= 2000) { k += 1 }
        j += 1 }
      i += 1 }
```

想看看Scala的for循环可能有多可怕？如果我们编译：
``` scala
object Test {
  def test() {}
}
```
我们得到以下bytecode：
``` scala
        [[syntax trees at end of cleanup]]// Scala source: test.scala
        package <empty> {
          final object Test extends java.lang.Object with ScalaObject {
            def test(): Unit = ();
            def this(): object Test = {
              Test.super.this();
              ()
            }
          }
        }
```

但是如果我们加了第一段代码到`test`方法中，我们就可以发现编译的结果中overhead是多么大：
``` scala
        [[syntax trees at end of cleanup]]// Scala source: test.scala
        package <empty> {
          final object Test extends java.lang.Object with ScalaObject {
            def test(): Unit = {
              var s$1: scala.runtime.IntRef = new scala.runtime.IntRef(0);
              scala.this.Predef.intWrapper(1).to(2000).foreach$mVc$sp({
                (new anonymous class Test$$anonfun$test$1(s$1): Function1)
              })
            };
            def this(): object Test = {
              Test.super.this();
              ()
            }
          };
          @SerialVersionUID(0) final <synthetic> class Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1$$anonfun$apply$mcVI$sp$2 extends scala.runtime.AbstractFunction1$mcVI$sp with Serializable {
            final def apply(k: Int): Unit = Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1$$anonfun$apply$mcVI$sp$2.this.apply$mcVI$sp(k);
            <specialized> def apply$mcVI$sp(v1: Int): Unit = Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1$$anonfun$apply$mcVI$sp$2.this.$outer .Test$$anonfun$$anonfun$$$outer().s$1.elem = Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1$$anonfun$apply$mcVI$sp$2.this.$outer .Test$$anonfun$$anonfun$$$outer().s$1.elem.+(1);
            <synthetic> <paramaccessor> private[this] val $outer: anonymous class Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1 = _;
            final <bridge> def apply(v1: java.lang.Object): java.lang.Object = {
              Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1$$anonfun$apply$mcVI$sp$2.this.apply(scala.Int.unbox(v1));
              scala.runtime.BoxedUnit.UNIT
            };
            def this($outer: anonymous class Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1): anonymous class Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1$$anonfun$apply$mcVI$sp$2 = {
              if ($outer.eq(null))
                throw new java.lang.NullPointerException()
              else
                Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1$$anonfun$apply$mcVI$sp$2.this.$outer = $outer;
              Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1$$anonfun$apply$mcVI$sp$2.super.this();
              ()
            }
          };
          @SerialVersionUID(0) final <synthetic> class Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1 extends scala.runtime.AbstractFunction1$mcVI$sp with Serializable {
            final def apply(j: Int): Unit = Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1.this.apply$mcVI$sp(j);
            <specialized> def apply$mcVI$sp(v1: Int): Unit = scala.this.Predef.intWrapper(1).to(2000).foreach$mVc$sp({
              (new anonymous class Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1$$anonfun$apply$mcVI$sp$2(Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1.this): Function1)
            });
            <synthetic> <paramaccessor> private[this] val $outer: anonymous class Test$$anonfun$test$1 = _;
            <synthetic> <stable> def Test$$anonfun$$anonfun$$$outer(): anonymous class Test$$anonfun$test$1 = Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1.this.$outer;
            final <bridge> def apply(v1: java.lang.Object): java.lang.Object = {
              Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1.this.apply(scala.Int.unbox(v1));
              scala.runtime.BoxedUnit.UNIT
            };
            def this($outer: anonymous class Test$$anonfun$test$1): anonymous class Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1 = {
              if ($outer.eq(null))
                throw new java.lang.NullPointerException()
              else
                Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1.this.$outer = $outer;
              Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1.super.this();
              ()
            }
          };                                                                                                                                                             
          @SerialVersionUID(0) final <synthetic> class Test$$anonfun$test$1 extends scala.runtime.AbstractFunction1$mcVI$sp with Serializable {
            final def apply(i: Int): Unit = Test$$anonfun$test$1.this.apply$mcVI$sp(i);
            <specialized> def apply$mcVI$sp(v1: Int): Unit = scala.this.Predef.intWrapper(1).to(2000).foreach$mVc$sp({
              (new anonymous class Test$$anonfun$test$1$$anonfun$apply$mcVI$sp$1(Test$$anonfun$test$1.this): Function1)
            });
            final <bridge> def apply(v1: java.lang.Object): java.lang.Object = {
              Test$$anonfun$test$1.this.apply(scala.Int.unbox(v1));
              scala.runtime.BoxedUnit.UNIT
            };
            <synthetic> <paramaccessor> val s$1: scala.runtime.IntRef = _;
            def this(s$1: scala.runtime.IntRef): anonymous class Test$$anonfun$test$1 = {
              Test$$anonfun$test$1.this.s$1 = s$1;
              Test$$anonfun$test$1.super.this();
              ()
            }
          }
        }
		
```
##结论
在scala没有解决[相关的bug #1138](https://issues.scala-lang.org/browse/SI-1338)前，还是小心用它的for循环，包括foreach。


##一点更新:
我在自己的工作站上测试了下面两端代码的运行时间：

三层循环，耗时7570毫秒：

``` scala
var s = 0
for (i <- 1 to 2000; 
     j <- 1 to 2000; 
     k <- 1 to 2000) 
  s += 1
```

一层循环，耗时1毫秒：

``` scala
var s = 0
for (i <- 1 to 2000*2000*2000) 
  s += 1
```
看来对于单层循环，scala的效率还可以接受。


