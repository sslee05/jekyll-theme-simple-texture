---
layout: post
title: "Cats Functor"
description: "cats functor"
categories: [scala-Cats]
tags: [scala,스칼라,Cats,cats, 켓츠, Functor, 함자]
redirect_from:
  - /2018/07/25/
---

> Cats Functor .
>

# Functor
functor에 관하여는 
[Functor](https://sslee05.github.io/blog/2017/09/10/scala-functor/)를 참조  

# Future 와 참조 투명성
Future는 부작용 계산에 대한 처리를 Future에서 할경우 예상치 못한 결과 즉 side-effect를 가질 수 있다.  
{% highlight scala %}
import scala.util.Random
import scala.concurrent.{Future,Await}
import scala.concurrent.ExecutionContext
import scala.concurrent.duration._

object FutureExplorer extends App {
  
  implicit val ec = ExecutionContext.global
  
  val future01 = {
    
    val r = new Random(0L)
    
    //부작용을 가지는 계산에 대한 처리 
    val x = Future(r.nextInt())
    
    for {
      a <- x
      b <- x
    }yield(a,b)
    
  }
  
  val future02 = {
    val r = new Random(0L)
    for {
      //참조 투명을 검증하기 위해 x에 Future을 대입
      a <- Future(r.nextInt()) 
      b <- Future(r.nextInt())
    }yield(a,b)
  }
  
  println(Await.result(future01, 1 seconds))
  println(Await.result(future02, 1 seconds))
  
}

//(-1155484576,-1155484576)
//(-1155484576,-723955400)
{% endhighlight %}

따라서 Future와 같은 성질의 다른 안전한 참조 투명성을 지닌 다른 유형인 찾아 보면 다음과 같이 Funtion 의 합성을 통하여 연산의 지연으로 Future 와 같은 미래의 연산을 나타 낼 수 있다.  

# Function
fn01: X => A, fn02: A => B 일때 fn01 compose fn02 이면  X => B 의 함수를 얻을 수 있다.  
{% highlight scala %}
val fn01: Int => Double = (i: Int) => i.toDouble
val fn02: Double => Double = (d: Double) => d * 2
val fn03: Double => String = (d: Double) => d.toString

//연산의 서술, 연산이 실제 실행이 되는 것이 아니다.
val fn04:Int => String = fn03 compose fn02 compose fn01

//연산의 실제 실행
println(fn04(5))
{% endhighlight %}

# Functor with Cats
## cats 에서의 functor 정의
{% highlight scala %}
import scala.language.higherKinds
trait Functor[F[_]] {
  def map[A,B](ma: F[A])(f: A => B):F[B]
}
{% endhighlight %}

## Functor의 law
[Functor](https://sslee05.github.io/blog/2017/09/10/scala-functor/)에도 언급이 되어 있다.  
Identity: 즉 Functor에 항등함수를 적용하면 원래의 Functor와 같다.  
Composition: 두 함수 f와 g로 합성후 map를 적용한 것은 함수 f를 map적용한 후 다음 함수 g를 map 적용한 것과 동일하다.
{% highlight scala %}
//Identity
fa.map(a => a) == fa

//Composition
fa.map(g(f(_))) == fa.map(f).map(g)
{% endhighlight %}



* Kramdown table of contents
{:toc .toc}


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
