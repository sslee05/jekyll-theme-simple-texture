---
layout: post
title: "Cats Monoid and Semigroup"
description: "cats Monoid 와 Semigroup"
categories: [scala-Cats]
tags: [scala,스칼라,Cats,cats, 켓츠]
redirect_from:
  - /2018/07/13/
---

> Cats 모노이드 와 세미그룹 .
>


* Kramdown table of contents
{:toc .toc}

일전에 [scala function](https://sslee05.github.io/blog/2017/09/09/scala-monoid/ "Monoid") 관련 post에서 Monoid에 관하여 기제한 적이 있다. 이를 보면 Cats의 Monoid and Semigroup은 이해하기 쉽다. 같은 내용이니 말이다. 다만 Cats에서 이를 어떻게 사용자 코드에 적용하는 지만 알면 된다.  

# Monoid & Semigroup
Monoid의 정의는 어떤 집합의 원소들에 대한 결합법칙을 만족하는 항등원과 이항연산이 있으며, 이 이항연산의 결과 또한 그 집합의 원소이다.  
그래서 Monoid정의시 다음과 같이 type class를 정의 했었다.  
{% highlight scala %}
trait Monoid[A] {
  def zero: A
  def op(a: A, b: A): A
}
{% endhighlight %}
Cats에서는 zero를 empty, op를 combine이라 부른다. 그래서 다음과 같이 정의 한다.  
{% highlight scala %}
trait Monoid[A] {
  def empty: A
  def combine(a: A, b: A): A
}
{% endhighlight %}
그런데 Cats 에서 어떠한 유형에 대하여는 항등원을 가질 수 없는 유형이 있기 때문에 다음과 같이 type class를 두었다.  
{% highlight scala %}
trait Semigroup[A] {
  def combine(a: A, b: A): A
}

trait Monoid[A] extends Semigroup[A] {
  def empty: A
}

object Monoid {
  def apply[A](implicit monoid: Monoid[A]) = monoid
}
{% endhighlight %}

## operator associativity & identity law
{% highlight scala %}
def associativeLaw[A](a: A, b:A, c:A)(implicit monoid: Monoid[A]): Boolean = 
  monoid.combine(a, monoid.combine(b, c)) ==  monoid.combine(monoid.combine(a,b),c) 
  
def identityLaw[A](a:A)(implicit monoid: Monoid[A]): Boolean = 
  (monoid.combine(a,monoid.empty) == a) && (monoid.combine(monoid.empty,a) == a)
{% endhighlight %}

# Monoid 만들어 보기 
Boolean type에 대한 and 연산자에 대한 monoid와 or연산자에 대한 monoid를 만들어 보자  
{% highlight scala %}
import cats._

object MonoidInstances extends App {
  
  implicit val orMonoid = new Monoid[Boolean] {
    def empty = false
    def combine(a: Boolean, b: Boolean): Boolean = a || b
  }
  
  implicit val andMonoid = new Monoid[Boolean] {
    def empty = true
    def combine(a: Boolean, b: Boolean): Boolean = a && b
  }
}
{% endhighlight %}

Set 에 대한 union 연산은 결합법칙이 성립하므로 다음과 같이 정의 할 수 있다.
{% highlight scala %}
implicit def setUnionMonoid[A] = new Monoid[Set[A]] {
  def empty = Set.empty[A]
  def combine(ma: Set[A], mb: Set[A]): Set[A] = ma union mb
}
{% endhighlight %}

하지만 차집합 연산인 intersect 는 결합법칙이 성립하지 않는다.  
그리고 empty 도 정의 할 수 없다.  
그러나 대칭차집합인 경우 즉 (A - B) uion (B - A ) 는  결합법칙이 성립한다. 즉 (A - B) - C == A - (B - C) 이 성립한다. 따라서 다음과 같이 정의 할 수 있다.
{% highlight scala %}
implicit def symmetriDiffSetMonoid[A] = new Monoid[Set[A]] {
  def empty = Set.empty[A]
  def combine(ma: Set[A], mb: Set[A]): Set[A] = 
    (ma diff mb) union (mb diff ma)
}
{% endhighlight %}

# Cats Monoid type class
Monoid 와 Semigroup 의 type class 는 cats.kernel.Monoid, cats.kernel.Semigroup에 정의 되어 있고 cats package에 type alias 로 선언 되어 있다.  
{% highlight scala %}
//type class alias
type Eq[A] = cats.kernel.Eq[A]
type PartialOrder[A] = cats.kernel.PartialOrder[A]
type Order[A] = cats.kernel.Order[A]
type Semigroup[A] = cats.kernel.Semigroup[A]
type Monoid[A] = cats.kernel.Monoid[A]
type Group[A] = cats.kernel.Group[A]

// object alias
val Eq = cats.kernel.Eq
val PartialOrder = cats.kernel.PartialOrder
val Order = cats.kernel.Order
val Semigroup = cats.kernel.Semigroup
val Monoid = cats.kernel.Monoid
val Group = cats.kernel.Group
{% endhighlight %}
위의 code는 cats pacakge object 에서 cats.kernel에 선언된 type class와 object에 대한 alias를 제공하고 있는 부분이다.  

Cats Kernel은 Cats의 작은 하위 project로 Cats의 full toolbox가 필요없는 가벼운 몇몇 핵심  type class들을 제공한다. 하지만 이들은 모두 cats pacakge에 alias 로 선언되어 있고 대부분은 이곳에 정의 되어 있다.  

# Cats Monoid instances
cats.instance에 해당하는 유형별로 type class instance 가 있으며 해당하는 type의 암시자가 있다면 그 암시자를 반환 하는 apply method가 Monoid object에 있다.  
{% highlight scala %}
import cats.Monoid
import cats.instances.string._
  
val ex01 = Monoid[String].combine("hellow"," world")
println(ex01)
  
import cats.instances.int._
val ex02 = Monoid[Int].combine(1, 2)
println(ex02)
{% endhighlight %}

# Cats Monoid interface syntax
combine method는 Semigroup의 method이며 Semigroup interface syntax에 |+| method를 제공하는 SemigroupOpt 생성 암시자가 있다. 따라서 다음과 같이 사용하면 된다.
{% highlight scala %}
import cats.instances.string._
import cats.instances.int._
import cats.syntax.semigroup._

val ex03 = "hellow" |+| " world"
println(ex03)
  
val ex04 = 1 |+| 2
println(ex04)
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
