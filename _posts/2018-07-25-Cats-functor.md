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
fn01: X => A, fn02: A => B 일때 fn01 compose fn02 이면  X => B 의 함수를 얻을 수 있다.  
{% highlight scala %}
val fn01: Int => Double = (i: Int) => i.toDouble
val fn02: Double => Double = (d: Double) => d * 2
val fn03: Double => String = (d: Double) => d.toString

//연산의 서술, 연산이 실제 실행이 되는 것이 아니다.
val fn04:Int => String = fn03 compose fn02 compose fn01

//이것은 Cats map를 빌리면 아래와 같이 표현할 수 있다.
import cats.instances.function._
import cats.syntax.functor._
val fn05: Int => String = fn01 map fn02 map fn03

//그리고 위의 것을 fn01, fn02, fn03자리를 익명함수로 치환하면
val fn:Int => String = 
  ((i: Int) => i.toDouble) map ( d => d * 2) map (d => d.toString)

//연산의 실제 실행
println(fn04(5))
//위의 실행결과와 같다.
println(fn(5))
{% endhighlight %}
위의 예제를 보면 어떤 단일 함수를 map으로 sequence하게 연결함으로써 어떤 연산의 chain을 만들어 낼 수 있으며 이는 단지 연산의 선언적 표현으로 실제 실행은 되지 않고 언제가 호출시 실행된다는 점에서 Future와 비슷하다고 할 수 있겠다.  
또 fn01,fn02,fn03을 익명함수로 치환해도 같은 결과를 얻을 수 있음을 알 수 있다. 이를 통해 Future와는 다르게 참조투명함를 알 수 있다.  

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

# Functor type class & instance
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

lift method도 지원된다.  
{% highlight scala %}
import cats.Functor
import cats.instances.option._

val fn = (i: Int) => i * 2
val opFn: Option[Int] => Option[Int] = Functor[Option].lift(fn)
println(opFn(Option(2)))
//Some(4)
{% endhighlight %}

# Functor syntax
Function 에는 map method가 없으며 대신 compose 혹은 andThen 이 있다.  
반면 List 나 Option 등은 내장된 map method가 있다.  
compiler는 내장 map method가 있을 경우 이를 먼저 찾으며, Function처럼 없다면 syntax가 하는 역할, 바로 확장 method를 찾을 것 이다.  

Funciton은 map method가 없다.
{% highlight scala %}
//not map method in Function
val fn01: Int => Int = (i: Int) => i * 2
val fn02: Int => Double = (i: Int) => i.toDouble
val fn03: Double => String = (d: Double) => d + "!"
  
val fn04: Int => String = fn03 compose fn02 compose fn01
println(fn04(5))
{% endhighlight %}

Functor syntax의 확장 class로 인해 내장 map method가 있는 것 처럼 사용 할 수 있다.  
{% highlight scala %}
import cats.instances.function._
import cats.syntax.functor._
  
val fn05: Int => String = fn01 map fn02 map fn03
println(fn05(5))
{% endhighlight %}
확장 method를 했다면 다음과 같을 것 이다.
{% highlight scala %}
import cats.Functor
import cats.syntax.functor._
def doMath[F[_]](ma: F[Int])(implicit F: Functor[F]): F[Int] = {
  ma.map(n => n * 2) //import cats.syntax.functor._  syntax 가 있어야 
}
{% endhighlight %}

cats.syntax.funtor 에 확장 class
{% highlight scala %}
import cats.Functor
implicit class FunctorOps[F[_], A](src: F[A]) {
  def map[B](func: A => B)(implicit functor: Functor[F]): F[B] =
    functor.map(src)(func)
}
{% endhighlight %}

# Custom type Functor
Option, Future , List 등 이들은 이미 Functor의 성질을 가지고 있다.  
Cats에서는 cats 의 type class인 Functor의 instance로 Option이나 Future등 이미 제공하고 있지만 아래의 예제를 통해 원하는 type의 Functor 구현을 보여 준다.  
{% highlight scala %}
import cats.Functor
  
implicit val optionFunctor = new Functor[Option] {
  def map[A,B](ma: Option[A])(f: A => B): Option[B] = ma map f
}
{% endhighlight %}

Future 인 경우 ExecutionContext의 암시자가 필요하므로 def 로 선언하며 암시자를 인자로 선언 해야 한다.  
{% highlight scala %}
import cats.Functor
import scala.concurrent.Future
import scala.concurrent.ExecutionContext

implicit def functorFunctor(implicit ec: ExecutionContext) = 
  new Functor[Future] {
    def map[A,B](ma: Future[A])(f: A => B): Future[B] = ma map f  
  }
{% endhighlight %}

# example
Tree에 대한 Functor를 만드는 예제 인데 기존에 했던 것이과 다른 것이 없다. 다만 주의 할 점은 variant에 대한 것이다. 이는 일명 smart constructor라는 (scala 빨간책 lazy부분에 나옴)것을 이용하면 된다.  
{% highlight scala %}
trait Tree[+A]

case class Branch[+A](left: Tree[A], right: Tree[A]) extends Tree[A]
case class Leaf[+A](value: A) extends Tree[A]

object Tree {
  def branch[A](left: Tree[A], right: Tree[A]): Tree[A] =
    Branch(left, right)

  def leaf[A](value: A): Tree[A] = Leaf(value)
}

import cats.Functor

implicit val treeFunctor = new Functor[Tree] {
  def map[A, B](ma: Tree[A])(f: A => B): Tree[B] = ma match {
    case Branch(left, right) => Branch(map(left)(f), map(right)(f))
     case Leaf(value)         => Leaf(f(value))
  }
}

val b01 = Tree.branch(Leaf(1), Leaf(2))
val b02 = Tree.branch(Leaf(3), Leaf(4))
val b03 = Tree.branch(b01, b02)

import cats.syntax.functor._
val bTree = b03.map(a => a + "!")
{% endhighlight %}
기존에 했던 것과 다를 것이 없다. 다만 varaint때문에 smart constructor를 하지 않는다면 명시적으로  b03 에 대한 명시적 type를 기제 해야 한다.


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
