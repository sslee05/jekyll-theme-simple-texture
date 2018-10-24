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

* Kramdown table of contents
{:toc .toc}

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
fn01: X => A, fn02: A => B 일때 fn01 compose fn02 이면  X => B 의 함수를 얻을 수 있다. 이를 통해 연산을 chain형태로 나타 낼 수 있으며 이때 이는 연산의 표현이며 표현이 곧 실행을 의미 하지는 않는다.  
{% highlight scala %}
val fn01: Int => Double = (i: Int) => i.toDouble
val fn02: Double => Double = (d: Double) => d * 2
val fn03: Double => String = (d: Double) => d.toString
  
//함수의 합성으로 연산의 chain 실제 실행 되지 않는다. 
val fn04: Int => String = fn03 compose fn02 compose fn01
  
//Cats 의 map 으로 표현 
import cats.instances.function._
import cats.syntax.functor._
  
val fn05:Int => String = fn01 map fn02 map fn03
  
// 함수의 변수 자리에 익명함수로 치환하여 참조투명성을 check 
val fn06:Int => String = 
 ((d:Double) => d.toString)  compose ((d:Double) => d * 2 ) compose ((i:Int) => i.toDouble)
val fn07:Int => String = 
  ((i:Int) => i.toDouble) map ((d:Double) => d * 2) map ((d:Double) => d.toString)
  
//실제 실행
println(fn04(5))
println(fn05(5))
println(fn06(5))
println(fn07(5))
{% endhighlight %}
위의 예제를 보면 어떤 단일 함수를 map으로 sequence하게 연결함으로써 어떤 연산의 chain을 만들어 낼 수 있으며 이는 단지 연산의 선언적 표현으로 실제 실행은 되지 않고 언제가 호출시 실행된다는 점에서 Future와 비슷하다고 할 수 있겠다.  

# Higher Kinded type 
## Higher Kined type(고계타입)
Functor, Monad, Applicative Functor 등을 이야기 할때 F\[_\] 이런 형태를 이야기 하지 않을 수 없다.  
우리가 java에서 List\<Integer\>, List\<String\> 등 Integer, String 등의 어떠한 정해지지 않은 유형을 담는 List를 표현할때  List\<A\> 라고 표현한다. Map은 Map\<K,V\> 이렇게.  
그럼 저런 List\<A\>, Map\<K,V\> 처럼 List나 Map 어떠한 유형을 내포하는 유형인데 List인지, Map 인지 정해지지 않은 유형을 표현할때는 어떻게 표현할까?  
이게 고계타입(Higher Kinded type)이다.  
일전에 Functor 게시글에서 이를 Higher Kinded type(고계타입 이라고)라고 하는데 이는 어떠한 type를 유형을 가지는 유형를 표현한 것이다.  


# cats Functor 
## type class
{% highlight scala %}
trait Functor[F[_]] {
  def map[A,B](ma: F[A])(f: A => B):F[B]
}

{% endhighlight %}

api에 보면  companion object의  apply method 은 아래와 과 같다.  
{% highlight scala %}
  def apply[F[_]](implicit functor: Functor[F]): Functor[F]
{% endhighlight %}
위의 코드에서 apply method가 적용될때는 F라는 고계타입에 해당하는 Functor가 암시적으로 받드시 있어야 한다. 따라서 import cats.instance.list._ 처럼 cats instance를 import해야 한다.  
{% highlight scala %}
import cats.Functor
import cats.instances.list._

//object Functor
//def apply[F[_]](implicit instance: Functor[F]): Functor[F]
val functor = Functor[List]
{% endhighlight %}

## Functor type class & instance
기존에 해왔듯이 Functor의 type class 도 같다. Functor type class와 companion object의 apply method가  있고 실제 기본 유형에 대한 instance들은 cats.instances에 있다.  
{% highlight scala %}
import cats.Functor
import cats.instances.list._
  
val xs = List(1,2,3,4,5)
val ys:List[Int] = Functor[List].map(xs)(n => n * 2)
println(ys)
//List(2, 4, 6, 8, 10)

import cats.instances.option._
  
val op01 = Option(123)
val op02:Option[Int] = Functor[Option].map(op01)(n => n * 2)
println(op02)
//Some(246)

{% endhighlight %}

## Functor syntax
위의 Function 의 예제 코드에서  function01 map function02 map function03 처럼 사용했다. 하지만 Function에는 map method가 없다.  
따라서 이것도 Function syntax가 관련될을 것이라 짐작 할 수 있다.  
{% highlight scala %}
import cats.Functor
implicit class FunctorOps[F[_], A](src: F[A]) {
  def map[B](func: A => B)(implicit functor: Functor[F]): F[B] =
    functor.map(src)(func)
}
{% endhighlight %}


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
