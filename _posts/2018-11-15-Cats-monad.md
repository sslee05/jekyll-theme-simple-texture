---
layout: post
title: "Cats Monad"
description: "cats monad"
categories: [scala-Cats]
tags: [scala,스칼라,Cats,cats, 켓츠, Monad, 모나드]
redirect_from:
  - /2018/11/15/
---

> Cats Monad .
>

* Kramdown table of contents
{:toc .toc}

# Monad
monad 관하여는 
[Monad](https://sslee05.github.io/blog/2017/09/11/scala-monad/)를 참조  

#  Cats Monad type class
Monad는 전의 글에서 Monad에 대하여 설명했다.  
여기서는 cats에서 어떻게 Monad를 쉽게 이용할 수 있는지를 살펴보자.  
우선 Monad의 type class는 아래와 같으며, Moand는 Functor이므로 아래와 같이 map 도 구현 가능 해야 한다.  
{% highlight scala %}
trait Monad[F[_]] {
 def pure[A](a: A): F[A]
 def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B]

 //Monad 는 Funtor이다 따라서 pure와 flatMap으로 map를 제공할 수 있엉야 한다.
 def map[A,B](ma: F[A])(f: A => B): F[B] = flatMap(ma)(a => pure(f(a)))
}
{% endhighlight %}
cats의 Monad type class는 아래와 같다.
{% highlight scala %}
trait Monad[F[_]] extends FlatMap[F] with Applicative[F]

//Left 항등법칙 
//pure(a).flatMap(f) == f(a)

//right 항등법칙 
//ma.flatMap(pure) == ma

//associativity 결합법칙
//ma.flatMap(f).flatMap(g) == ma.flatMap(a => f(a).flatMap(g))
{% endhighlight %}
위에의 코드를 보면 FlatMap에서 flatMap를  Applicative fuctor에서 pure, map method를 상속 받는다.  

# Cats Monad instance
{% highlight scala %}
//Monad[F[_]] extends FlatMap[F] with Applicative[F]
// FlatMap으로 부터 flatMap method
// Applicative으로 부터 pure, map method
import cats.Monad
import cats.instances.list._
import cats.instances.option._ //for Monad

val opt1 = Monad[Option].pure(3)
val opt2 = Monad[Option].flatMap(opt1)(a => Some(a + 2))
val opt3 = Monad[Option].map(opt2)(a => a * 100)

val list1 = Monad[List].pure(3)
val list2 = Monad[List].flatMap(List(1,2,3))(a => List(a, a * 10))
val list3 = Monad[List].map(list2)(a => a + 123)
{% endhighlight %}

Future인경우 implicit 가 필요한데 ExecutionContext 이것이 scope에 없으면  
cats.instances.future를 해도 implicit Monad\[Future\]가 없다고 한다.
{% highlight scala %}
import scala.concurrent.Future
import cats.instances.future._
import scala.concurrent.ExecutionContext.Implicits.global

val futureMonad: Monad[Future] = Monad[Future]
val future: Future[Int] = Monad[Future].pure(1)
{% endhighlight %}

# Cats Monad Syntax
3개의 cats.syntax에서 얻는다.  
 - cats.syntax.flatMap : flatMap 의 사용   
 - cats.syntax.functor : map 의 사용  
 - cats.syntax.applicative : pure 의 사용  
3개을 모두 사용 할경우 일일이 하지 않고 다음과 같이 하면 한번에  
import cats.implicits  
{% highlight scala %}
import cats.syntax.applicative._
val op4: Option[Int] = 1.pure[Option]
val rs: List[Int] = 1.pure[List]
{% endhighlight %}
flatMap이나 map같은 경우는 pure 와 같이 쉽게 할 수 가 없다.  
왜냐면 type에 맞는 연산이 정의된 implicit가 없기 때문이다.  
{% highlight scala %}
import cats.syntax.flatMap._ //for flatMap
import cats.syntax.functor._ //for map
def sum[F[_]: Monad](ma: F[Int], mb: F[Int]): F[Int] =
  ma.flatMap(a => mb.map(b => a * b))

val r = sum(Monad[Option].pure(2), Option(5))
println(r)
//Some(10)
val rs2:List[Int] = sum(List(1,2,3), Monad[List].pure(5))
println(rs2)
//List(5, 10, 15)
{% endhighlight %}
위에서 sum(1,2)는 작동하지 않는다. 이런 경우 type alias를 주면 해결 할 수 있는데 전에 [Traversable](https://sslee05.github.io/blog/2017/09/17/scala-traverse-functor/) 예에서 traverse method를 가지고 map method를 만들때 Applicative가 필요 했는데 이때 type Id\[A\] = A를 이용한 적이 있었다. 다만 cats에서 이미 이에 대한 type 를 만들어 놓은 것을 import 하면 된다.  
{% highlight scala %}
//now work sum(1,2)
/ type Id[A] = A
// cats에서 Id 제공
import cats.Id
import cats.instances.int._
val rs3:Int = sum(3: Id[Int], 5: Id[Int])
println(rs3)

val ex01:Id[Int] = 1
val ex02:Id[String] = "abcd"
val rs4:Int = Monad[Id].pure(2).flatMap(i => i * 5)
val rs5:Id[Int] = Monad[Id].pure(2).flatMap(i => i * 5)

import cats.syntax.eq._
println(rs4 === rs5)
{% endhighlight %}

# cats Either syntax 유용용한 method
## asRight[LeftT], asLeft[RightT]
아래의 코드는 compile 되지 않는다.  
이유는 접기 결과 유형이 Either가 아닌 Right로 타입 추론이 되면서 Left가 문제가 된다.
또 Right의 Left type parameter를 Nothing으로 추론 된다.  
{% highlight scala %}
val xs = List(1,2,3,4,5)
xs.foldLeft(Right(0))((acum,i) => 
 if(i < 3) acum.map(n => n + 1) else Left("Not Supported"))
{% endhighlight %}
따라서 compile error가 나지 않게 하기 위해서 다음과 같이 하면 된다.  
{% highlight scala %}
val xs = List(1,2,3,4,5)
xs.foldLeft[Either[String,Int]](Right(0))((acum, i) 
 => if(i < 3) acum.map(n => n + i) else Left("Not Supported"))
{% endhighlight %}
이를 좀더 쉽게 cats의 asRight[LeftT], asLeft[RightT]를 사용하면 같은 결과를 가질 수 있다.
{% highlight scala %}
import cats.syntax.either._
  // asRight[LeftT]  asLeft[RightT]
xs.foldLeft(0.asRight[String])((acum, i) => 
	if(i < 3) acum.map(n => n + i) else "Not Supprted".asLeft[Int])
// Either[String,Int]
{% endhighlight %}

## cats.syntax.either.EitherObjectOps
cats.syntax.either 에는 Either companion object type를 받는 method들이 있다.
1. catchOnly\[A\], catchNonFatal\[A\]
{% highlight scala %}
val rs03 = Either.catchOnly[NumberFormatException]("foo".toInt)
println(rs03)
//Left(java.lang.NumberFormatException: For input string: "foo")

//def catchNonFatal[A](f: => A): Either[Throwable, A]
val rs04 = Either.catchNonFatal(sys.error("Occur error"))
println(rs04)
//Left(java.lang.RuntimeException: Occur error)
{% endhighlight %}

2. fromTry\[A\](t: Try\[A\]): Either\[Throwable, A\]
{% highlight scala %}
val rs05 = Either.fromTry(Try("foo".toInt))
println(rs05)
//Left(java.lang.NumberFormatException: For input string: "foo")
{% endhighlight %}

3. def fromOption\[A, B\](o: Option\[B\], ifNone: => A): Either\[A, B\]
{% highlight scala %}
val rs06 = Either.fromOption[String,Int](None, "Oops!")
println(rs06)
//Left(Oops!)
{% endhighlight %}

## Transforming Either
cats.syntax.ethier 에는 변환 method들이 많다.  
1. orElse, getOrElse
{% highlight scala %}
//1. orElse, 2. getOrElse
val rs07 ="Error".asLeft[Int].getOrElse(0)
println(rs07) //0
val rs08 = "Error".asLeft[Int].orElse(3.asRight[String])
println(rs08) //Right(3)
{% endhighlight %}

2. ensure Right의 조건 검증
{% highlight scala %}
val rs09 = -1.asRight[String].ensure("must not be negative value")(i => i >= 0)
println(rs09)
//Left(must not be negative value)
{% endhighlight %}

3. recover, recoverWith
{% highlight scala %}
val rs10 = "Error".asLeft[Int].recover{
  case _: String => -1
}
println(rs10)
//-1

val rs11 = "Error".asLeft[Int].recoverWith {
  case _: String => Right(-1)
}
println(rs11)
//Right(-1)
{% endhighlight %}

4. leftMap, bimap
{% highlight scala %}
val rs12 = "leftMap".asLeft[Int].leftMap(_.reverse)
println(rs12)//Left(paMtfel)

val rs13 = 5.asRight[String].bimap(msg => msg.reverse, i => i * 10)
val rs14 = "leftMap".asLeft[Int].bimap(ms => ms.reverse, i => i * 10)
println(rs13)//Right(50)
println(rs14)//Left(paMtfel)
{% endhighlight %}

5. swap
{% highlight scala %}
//swap
val rs15 = "foo".asLeft[Int].swap
println(rs15)//Right(foo)
{% endhighlight %}

# Eval Monad
## 평가
eager    : 즉시 평가(값으로서)  
lazy     : 실행시 평가  
memoized : 실행시 평가 이후 cache  
{% highlight scala %}
//vals = eager + memoized
val x = {
  println("Computing X")
  math.random()
}
//Computing X
//x: Double = 0.7810079442262585

println(x)
//0.7810079442262585
println(x)
//0.7810079442262585
println(x)
//0.7810079442262585

//def = lazy + not memoized
def y = {
  println("Computing Y")
  math.random()
}
//y: Double
println(y)
//Computing Y
//0.5771256769413449
println(y)
//Computing Y
//0.21990502916453714
println(y)
//Computing Y
//0.4958650011258595

//lazy vals = lazy + memoized
lazy val z = {
  println("Computing Z")
  math.random()
}
//z: Double = <lazy>
println(z)
//Computing Z
//0.614124247006023
println(z)
//0.614124247006023
println(z)
//0.614124247006023
{% endhighlight %}

cats에서 Now 는 즉시평가(val과 같이),  
Always는 실행시 매번 평가(def 같이),  
Later는 처음 실행시 평가 이후 cached처럼(momoized 같이) 작동 한다.
{% highlight scala %}
import cats.Eval
val cx = Eval.now(math.random())
//cx: cats.Eval[Double] = Now(0.7542159377020204)
val cy = Eval.always(math.random())
//cy: cats.Eval[Double] = cats.Always@3cc198f7
val cz = Eval.later(math.random())
//cz: cats.Eval[Double] = cats.Later@531d6244

println(cx.value)//0.7542159377020204
println(cy.value)//0.5143690057772041
println(cz.value)//0.012411074690963253

println(cx.value)//0.7542159377020204
println(cy.value)//0.5093222535099888
println(cz.value)//0.012411074690963253
{% endhighlight %}

## Eval Monad
{% highlight scala %}
import cats.Eval
val greeting: Eval[String] = Eval.always{ println("Step 01"); "Hellow";}.map{s => println("Step 02"); s"$s world";}

println(greeting.value)
println(greeting.value)
/* 결과
start evaluation
Step 01
Step 02
Hellow world
Step 01
Step 02
Hellow world
*/
{% endhighlight %}
위에 결과를 보면 always는 def 처럼 평가 됨을 알 수 있다. 
{% highlight scala %}
val ans: Eval[Int] = for {
  a <- Eval.now{ println("Step01"); 10} //바로 평가 val 즉 값처럼 
  b <- Eval.always{println("Step02"); 5}
} yield {
  println("Add A and B");
  a + b
}

println("start evaluation")
println(ans.value)
println(ans.value)

/*결과
Step01 //선언과 동시에 출력
start evaluation
Step02
Add A and B
15
Step02
Add A and B
15
*/
{% endhighlight %}
위에 결과를 보면 평가전에 Step01 이 찍히는 것으로 봐서 선언과 동시에 값으로서 평가가 되어 진후 아래의 사용할 때 step01은 찍히지 않는 것을 보아 다시 평가 하지 않는다.  
{% highlight scala %}
val saying = Eval.always{ println("Step01"); "The cat"}
  .map{s => println("Step02"); s" $s sat on"}.memoize
  .map{s => println("Step03"); s"$s the mat"}

println("start evaluation")
println(saying.value)
println(saying.value)

/*결과
start evaluation
Step01
Step02
Step03
 The cat sat on the mat
Step03 //memoize 로 인하여 위의 연산 chain결과는 cache처럼 
 The cat sat on the mat
*/
{% endhighlight %}
위의 예제에서 memoize 로 인하여 앞에서의 연산chain이 한번 평가 이후 2번째 부터는 cahce처럼 행동  

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
