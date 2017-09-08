---
layout: post
title: "scala monoid"
description: "scala monoid"
categories: [language]
tags: [scala]
redirect_from:
  - /2017/09/08/
---

> scala monoid.
>
> 이 내용은 저자 Paul Chiusano가 쓴 "Functional Programming in scala" 책을 공부하며 정리한 것 임.
> (잘 못 이해한 것이 있을 수 있음)

* Kramdown table of contents
{:toc .toc}

# 수학적 기본 지식

## 대수란
정수나 유리수와 같이 우리에게 익숙한 수들의 기본 __성질__ 들을 추상화, 일반화하는 것을 포함하는 
아이디어와 기능으로 구성된 수학적 세계 중에 하나.

## 대수학 
정수가 가지고 있는 __성질__ 의 일반화에 대한 연구이다.

## 군
__결합법칙__, __항등법칙__ , __교환법칙__ 이 성립하는 성질을 임의의 집합으로 일반화 한 것이 군이다.
__✶__ 를 하나의 집합 G에 대한 __이항 연산__ 이라 한다면( 임이의 x,y ∈ G => x ✶ y ∈ G )
다음의 성질을 만족하면 __G__ 또는 __(G,✶)__ 를 군이라 한다.
   l-1  ✶에 대한 결합법칙 ) x,y,z ∈ G => x ✶ (y ✶ z) = (x ✶ y) ✶ z)   
   l-2 항등원과 ✶에 대한 항등법칙) 모든 x ∈ G 에 대하여 e ✶ x = x ✶ e = x 을 만족하는 e ∈ G 가 존재한다.
   l-3 x는 역원를 갖는다) 각각의 x ∈ G 에 대하 x^-1 ∈ G 가 존재하며 x ✶ x^-1 = x^-1 ✶ x = e (.)

## 사상
대상의 관계에 대한 mapping, 혹은 function ( 임이의 군 __=>__ 임이의 군 )


# 간단한 예제작성 해보기

lib 를 사용하면 되지만 여기서는 lib 설계 방향을 배우는 것 임으로 만들어 가는 과정을 보자.
여기서는 동시성 프로그램을 하는 lib를 만드는 것을 목표로 했을 경우의 예이다.
{% highlight scala %}
val xs = List(1,2,3,4,5)
val rs = xs.foldLeft(0)((x,y) => x + y)
println(rs)
{% endhighlight %}

위의 예는 List의 원소의 sum을 구하는 것을 foldLeft를 이용하였다.
이를 동시에 나누어서 하려면 어떻게? 간다한 예부터 만들면

{% highlight scala %}
def sum(xs: IndexedSeq[Int]): Int = {
    if(xs.size  <= 1) xs.headOption getOrElse 0
    else {
        val (l,r) = xs.splitAt(xs.length / 2)
        sum(l) + sum(r)
    }
}
{% endhighlight %}

위의 예제는 이제 노리적으로 left와 right를 분리하여 처리 할 수 있게 되었다.
(물론 실제적으로는 left오 right가 동시에 처리 되진 않지만)

# 자료형식으로 부터 도출 되어야 하는 필요한 함수 도출
sum(l)와 sum(r)를 보고 병렬처리를 나타내는 자료형식이 필요함을 도출 또 그에 따른 그 자료형식을 만들어 반환 하는 함수와, 병렬처리 결과를 얻는 함수가 필요함을 알 수 있다.
{% highlight scala %}
trait Par[+A]

object Par {
  //병렬처리할 대상을 지연으로 받아 평가할 수 있는 계산을 돌려 준다.
  def unit[A](a: => A): Par[A] = ???
  //병렬 계산에서의 결과 값을 추출 한다.
  def get[A](par: Par[A]): A = ???
}
{% endhighlight %}

위의 code을 자기고 다시 sample code를 보자
{% highlight scala %}
def sum(xs: IndexedSeq[Int]): Int = {
    if(xs.size  <= 1) xs.headOption getOrElse 0
    else {
        val (l,r) = xs.splitAt(xs.length / 2)
        val rl = Par.unit(sum(l))
        val rr = Par.unit(sum(r))

        Par.get(lr) + Par.get(rr)
    }
}
{% endhighlight %}

# 문제점 찾기
위의 sample code를 보면 Par.unit(sum(l)) 과 Par.unit(sum(r)) 로 부터 논리적으로
(각각 병렬처리가 된다고 생각하고) 동시에 처리리는 되겠지만 get 함수에 의하여 결국 left이 결과를 얻기 위해 기다리면서 right는 대기하게 된다. 이름 참조치환으로 해보면 명백히 알 수 있다.
{% highlight scala %}
Par.get(Par.unit(sum(l))) + Par.get(Par.unit(sum(r)))
{% endhighlight %}

# 해결점 찾기
get 를 호출 하지 않게 하려면 어떻게 ?
map를 만들어 보자.
{% highlight scala %}
trait Par[+A]

object Par {
  //병렬처리할 대상을 지연으로 받아 평가할 수 있는 계산을 돌려 준다.
  def unit[A](a: => A): Par[A] = ???
  //병렬 계산에서의 결과 값을 추출 한다.
  def get[A](par: Par[A]): A = ???
  //처리결과를 조합한다.
  def map2[A,B,C](a: Par[A], b: Par[B])(f: (A,B) => C): Par[C] = ???
}
{% endhighlight %}

적용해보자 이때 sum의 return type은 Par[Int]이 될 것이다.
{% highlight scala %}
def sum(xs: IndexedSeq[Int]): Par[Int] = {
    if(xs.size  <= 1) xs.headOption getOrElse 0
    else {
        val (l,r) = xs.splitAt(xs.length / 2)
        val rl = Par.unit(sum(l))
        val rr = Par.unit(sum(r))

        Par.map2(rl,rr)(_ + _)
    }
}
{% endhighlight %}

# 다시 문제점 찾기
생가해 보면 map2(left,right) 에서 목록의 left가 다 소진 되어야 right가 처리됨을 알 수 있다.
왜냐면 scala 의 function parameter는 엄격(strictness)하기 때문에 parameter의 결과가 목록의 인자로 넘어가기 때문에 목록의 left의 depth까지 소진 map2 처리 후 최초의 left,right가 처리 될테니까.

그럼 map2의 parameter를 laziness 하게 바꾸어야 한다. 하지만 그렇게 된다면 모든 map2의 호출 client는 목록이 작은 경우에도 선택없이 항상 병렬처리 리소스를 하용해야 한다.

# 명시적 분기
병렬처리의 결과들의 조합되어야 하는 것과 병렬처리 여부를 분리한다. 따라서 map2의 인자는 strictness하게 두고 명시적으로 lazy하게 인자를 받아들이는 함수를 만들자.
{% highlight scala %}
trait Par[+A]

object Par {
  //병렬처리할 대상을 지연으로 받아 평가할 수 있는 계산을 돌려 준다.
  def unit[A](a: => A): Par[A] = ???
  //병렬 계산에서의 결과 값을 추출 한다.
  def get[A](par: Par[A]): A = ???
  //처리결과를 조합한다.
  def map2[A,B,C](a: Par[A], b: Par[B])(f: (A,B) => C): Par[C] = ???
  //명시적으로 선언함으로써 clinet는 병렬처리를 처리하고 싶을때 사용할 수 있다.
  def fork[A](a: => Par[A]): Par[A] = ???
}
{% endhighlight %}

{% highlight scala %}
def sum(xs: IndexedSeq[Int]): Par[Int] = {
    if(xs.size  <= 1) xs.headOption getOrElse 0
    else {
        val (l,r) = xs.splitAt(xs.length / 2)
        Par.map2(fork(sum(l)),fork(sum(r)))(_ + _)
    }
}
{% endhighlight %}
이제 map2의 left의 인자 depth 처리가 끝나지 않아도 최상위 left와 right가 동시에 처리 된다.

# Practice
* Par (Parallel) object 만들기
* Par 의 type을 정의 하라(ExecutorService를 받아 병렬계산 서술을 나타내는)
* java.util.concurrent.Future 를 상속한 UnitFuture를 Par object에 구현하라.
* 상수값을 받아 병렬 계산으로 승격하는 함수를 작성하라.
{% highlight scala %}
def unit[A](a: A): Par[A] = ???
{% endhighlight %}

* Par object에 주어진 인수가 동시적으로 평가될 계산임을 표시하는 병렬분기 처리을 명시하는 함수를 작성하라
{% highlight scala %}
def fork[A](a: => Par[A]): Par[A] = ???
{% endhighlight %}

* Par object에 평가되지 않는 인수를 Par로 감싸고, 그것을 병렬 평가 대상으로 표시하는 다음 함수를 작성하라.
{% highlight scala %}
def lazyUnit[A](a: => A): Par[A] = ???
{% endhighlight %}

* Par object에 계산을 실제로 실행서 Par로 부터 값을 추출하는 다음의 함수를 작성하라.
{% highlight scala %}
def run[A](es:ExecutorService)(par: Par[A]): Future[A] = ???
{% endhighlight %}

* Par object에 두 병렬처리의 결과를 받아 함수를 적용하는 다음의 함수를 작성하라.
{% highlight scala %}
def map2[A,B,C](parA: Par[A], parB: Par[B])(f: (A,B) => C): Par[C] = ???
{% endhighlight %}

* Par object에 함수 f: A => B 를 비동기적으로 평가되게 적용하는 함수를 작성하라.
{% highlight scala %}
def asyncF[A,B](f: A => B): A => Par[B] = ???
{% endhighlight %}

* Par object에 함수 다른 병렬처리를 받아 f: A => B 를 적용하여 승급시키는 함수 map를 작성하라.
{% highlight scala %}
def map[A,B](parA: Par[A])(f: A => B): Par[B] = ???
{% endhighlight %}

* map 함수를 이용하여 Par object에 병렬계산 목록의 결과가 정렬된게 변환하는 다음의 함수를 작성하라.
{% highlight scala %}
def sortPar(parList: Par[List[Int]]): Par[List[Int]] = ???
{% endhighlight %}

* Par object에 sequence 함수를 작성하라.
{% highlight scala %}
def sequence[A](xs: List[Par[A]]): Par[List[A]] = ???
{% endhighlight %}

* Par object에 처리대상 목록을 n개의 병렬계산으로 처리하고 그 결과를 기다렸다가 결과을 하나의 목록으로 나타내는 다음의 함수를 구현하라.
{% highlight scala %}
def parMap[A,B](xs: List[A])(f: A => B): Par[List[B]] = ???
{% endhighlight %}

* Par object에 목록의 요소를 병렬로 걸러내는 다음 함수를 구현하라.
{% highlight scala %}
def parFilter[A](xs: List[A])(p: A => Boolean): Par[List[A]] = ???
{% endhighlight %}

* Par object에 다음의 조건 cond: Par[Boolean] 의 결과가 true이면 t:Par[A] 를 사용하여 계산을 진행하고 false이면 f:Par[A] 를 사용하는 다음의 함수를 구현하라.
{% highlight scala %}
def choice[A](cond: Par[Boolean])(t: Par[A], f: Par[A]): Par[A] = ???
{% endhighlight %}

* Par object에 위의 choice를 좀더 일반화 해서 어떤 조건에 기초해서 N개의 계산중 하나를 선택해서 계산하는 다음의 함수를 구현하라.
{% highlight scala %}
def choiceN[A](n: Par[Int])(xs: List[Par[A]]): Par[A] = ???
{% endhighlight %}

* Par object에 위의 choice함수를 목록이 아닌 collection 의 Map 에서 하나를 선택해서 계산하는 다음의 함수를 구현하라.
{% highlight scala %}
def choiceMap[K,V](key: Par[K])(choices: Map[K,Par[V]]): Par[V] = ???
{% endhighlight %}

* 위의 choice,choiceN,choiceMap를 보면 좀 더 일반화 할 수 있다. 첫 계산이 준비되기도 전에 둘째 계산이 반드시 존재할 필요가 없다.List 나 Map에 둘째 계산이 저장될 필요가 없다. 첫째 계산의 결과를 이용해서 둘째 계산를 즉시 생성할 수도 있다. 이러한 함수를 보통 flaMap으로 칭한다. Par object에 다음의 flatmap함수를 구현하라.
{% highlight scala %}
def flatMap[A,B](par: Par[A])(f: A => Par[B]): Par[B] = ???
{% endhighlight %}

* Par object에 choice,choiceN,choiceMap를 flatMap를 이용하여 재정의 하라.

* 위의 flatMap보면 2단계로 분리 할 수 있다. f: A => Par[B] 를 적용. 그 결과 Par[Par[A]]를 flatten 하게 만든다. 이 flatten 함수를 따로 만들어 다른 대수적 lib에 사용 할 수 있다. Par object에 다음의 함수를 작성하라.
{% highlight scala %}
def join[A](a: Par[Par[A]]): Par[A] = ???
{% endhighlight %}

* Par object에 flatMap를 join를 이용하여 재정의 하라.
* Par object에 join를 flatMap을 이용하여 재정의 하라.
* Par object에 map2를 flatMap과 map를 이용하여 재정의 하라.(map2와 차이점을 파악하라)

# 문제 결과 (잘못 된 것이 있을 수 있음)
{% highlight scala %}
package basic.parallel

import java.util.concurrent.Callable
import java.util.concurrent.ExecutorService
import java.util.concurrent.Executors
import java.util.concurrent.Future
import java.util.concurrent.TimeUnit


object Par {

  type Par[A] = ExecutorService => Future[A]

  private case class UnitFuture[A](get: A) extends Future[A] {
    def isDone = true
    def get(timeout: Long, units: TimeUnit): A = {
      println("unitFuture in get function")
      get
    }
    def isCancelled = false
    def cancel(evenIfRunnig: Boolean): Boolean = false
  }

  def fork[A](a: => Par[A]): Par[A] = es => {
    es.submit(new Callable[A]{
      def call = {
        val f = a(es)
        f.get
      }
    })
  }

  def run[A](es:ExecutorService)(par: Par[A]): Future[A] = par(es)

  def unit[A](a: A): Par[A] = es => UnitFuture(a)

  def lazyUnit[A](a: => A): Par[A] = fork(unit(a))

  def asyncF[A,B](f: A => B): A => Par[B] = a => lazyUnit(f(a))

  def map2[A,B,C](parA: Par[A], parB: Par[B])(f: (A,B) => C): Par[C] = es => {
    val a = parA(es)
    val b = parB(es)
    UnitFuture(f(a.get,b.get))
  }

  def map[A,B](parA: Par[A])(f: A => B): Par[B] = 
    map2(parA,unit(()))((a,b) => f(a))

  def sortPar(parList: Par[List[Int]]): Par[List[Int]] = 
    map(parList)(a => a.sorted)

  def sequence[A](xs: List[Par[A]]): Par[List[A]] = 
    xs.foldRight[Par[List[A]]](unit(Nil))( (a,b) => map2(a,b)( (a1,b1) => a1 :: b1 ))

  def parMap[A,B](xs: List[A])(f: A => B): Par[List[B]] = fork {
    val rs:List[Par[B]] = xs.map(asyncF(f))
    sequence(rs)
  }

  def parFilter[A](xs: List[A])(p: A => Boolean): Par[List[A]] = {
    val rs:List[Par[List[A]]] = xs.map(asyncF[A,List[A]]((x:A) => if((p(x))) List(x) else Nil ))
    map(sequence(rs))(x => x.flatten)
  }

  def choice[A](cond: Par[Boolean])(t: Par[A], f: Par[A]): Par[A] = es => {
    if(run(es)(cond).get) t(es)
    else f(es)
  }

  def choiceN[A](n: Par[Int])(xs: List[Par[A]]): Par[A] = es => {
    val rn = run(es)(n).get
    run(es)(xs(rn))
  }

  def choiceMap[K,V](key: Par[K])(choices: Map[K,Par[V]]): Par[V] = es => {
    val k = run(es)(key).get
    run(es)(choices(k))
  }

  def flatMap[A,B](par: Par[A])(f: A => Par[B]): Par[B] = es => {
    val av = run(es)(par).get
    run(es)(f(av))
  }

  def choiceViaFlatMap[A](cond: Par[Boolean])(t: Par[A], f: Par[A]): Par[A] = 
    flatMap(cond)(x => if(x) t else f)

  def choiceMaoViaFlatMap[K,V](key: Par[K])(choices: Map[K,Par[V]]): Par[V] = 
    flatMap(key)(x => choices(x))

  def join[A](a: Par[Par[A]]): Par[A] = es => {
    run(es)(run(es)(a).get())
  }

  def flatMapViaJoin[A,B](par: Par[A])(f: A => Par[B]): Par[B] = 
    join(map(par)(f))

  def joinViaFlatMap[A](a: Par[Par[A]]): Par[A] = 
    flatMap(a)(x => x)

  // map2와는 달리 parA와 parB는 동시에 실행되지 않는다.
  def map2ViaFlatMap[A,B,C](parA: Par[A], parB: Par[B])(f: (A,B) => C): Par[C] = 
    flatMap(parA)(a => map(parB)(b => f(a,b)))
}

object ParDriver extends App {
  import basic.parallel.Par._
  val es = Executors.newFixedThreadPool(5)
  val xs = List(5,4,3,6,1)
  println(xs(2))
  println(parFilter(xs)(x => x > 3)(es).get)
}
{% endhighlight %}


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture