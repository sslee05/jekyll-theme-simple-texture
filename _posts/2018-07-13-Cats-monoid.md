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


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
