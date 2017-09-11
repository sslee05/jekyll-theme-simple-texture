---
layout: post
title: "scala monoid practice"
description: "scala monoid practice"
categories: [language]
tags: [scala-practice]
redirect_from:
  - /2017/09/08/
---

> scala monoid.  
> (Paul Chiusano 와 Runar Bjarnason이 쓴 Functional Programming in scala 책에서)
>


* Kramdown table of contents
{:toc .toc}

# Monoid 란
### trait Monoid를 작성하라
{% highlight scala %}
trait Monoid[A] {
  def op(a: A, b: A): A = ???
  def zero: A = ???
}
{% endhighlight %}

### String + 연산에 대한 Monoid를 만들어 만들어라.
{% highlight scala %}
def stringMonoid: Monoid[String] = ???
{% endhighlight %}

### Int + 연산에 대한 Monoid를 만들어 만들어라.
{% highlight scala %}
def intAddMonoid: Monoid[Int] = new Monoid[Int] = ???
{% endhighlight %}

### Int * 연산에 대한 Monoid를 만들어 만들어라.
{% highlight scala %}
def intProductMonoid: Monoid[Int] = new Monoid[Int] = ???
{% endhighlight %}

### Boolean or 연산에 대한 Monoid를 만들어라.
{% highlight scala %}
def booleanOr: Monoid[Boolean] = new Monoid[Boolean] = ???
{% endhighlight %}

### Boolean and 연산에 대한 Monoid를 만들어라.
{% highlight scala %}
def booleanAnd: Monoid[Boolean] = new Monoid[Boolean] = ???
{% endhighlight %}

### Option 조합 연산에 대한 Monoid를 만들어라.
{% highlight scala %}
def optionMonoid[A]: Monoid[Option[A]] = new Monoid[Option[A]] = ???
{% endhighlight %}

### 자기함수의 조합 연산애 대한 Monoid를 만들어라.
{% highlight scala %}
def endoMonoid[A]: Monoid[A => A] = new Monoid[A => A] = ???
{% endhighlight %}

### foldMap를 구현
{% highlight scala %}
def foldMap[A,B](xs: List[A])(f: A => B)(m: Monoid[B]): B = ???
{% endhighlight %}

### foldMap 함수를 foldRight를 이용하여 구현하라
{% highlight scala %}
def foldMap[A,B](xs: List[A])(f: A => B)(m: Monoid[B]): B = ???
{% endhighlight %}

### foldRight를 foldMap을 이용해서 구현하라.
{% highlight scala %}
def foldRightViaFoldMap[A,B](xs:List[A])(z:B)(f:(A,B) => B): B = ???
{% endhighlight %}

### foldLeft를 foldMap을 이용해서 구현하라.
정의역과 치역을 바꾸는 함수가 필요하다. 이를 구현 하라. 
{% highlight scala %}
def dualMonoid[A](m: Monoid[A]) = new Monoid[A] = ???
{% endhighlight %}

{% highlight scala %}
 def foldLeftViaFoldMap[A,B](xs: List[A])(z:B)(f: (B,A) => B): B = ???
{% endhighlight %}

# 결합법칙과 병렬성 
### IndexedSeq에 대한 foldMap를 구현하라.
구현은 반드시 순차열을 둘로 분할해서 재귀적으로 각 절반을 처리하고 그 결과들을 monoid를 이용해서 결합해야 한다.
{% highlight scala %}
def foldMapV[A,B](v: IndexedSeq[A], m: Monoid[B])(f: A => B): B = ???
{% endhighlight %}

### foldMap의 병렬버전도 구현해라.
parallel chapter에서 foldMap를 만들었다 이를 monoid를 이용해서 구현하라. 이때  
승격 함수인 Monoid[A] => Monoid[Par[A]] 가 필요하다.
{% highlight scala %}
import basic.parallel.Par._
def par[A](m: Monoid[A]): Monoid[Par[A]] =???
def parFoldMap[A,B](v: IndexedSeq[A], m: Monoid[B])(f: A => B): Par[B] = ???
{% endhighlight %}

### foldMap를 이용해서 주어진 IndexedSeq[Int] 가 정렬되어 있는지 점검하라.
Monoid를 이용할 것
{% highlight scala %}
def ordered(xs: IndexedSeq[Int]): Boolean = ???
{% endhighlight %}

# 접을 수 있는 자료구조
### 여러 자료구조에서 fold의 공통점을 뽑아 Foldable trait를 구현하라.
앞 chapter에서 List,Option,Par,Tree 등에 접기 fold가 함수 들이 있었다 이들은 매개변수 유형만  
다를 뿐이다 이들의 공통점을 뽑으면 다음과 같은 trait로 일반화 할수 있다. 이를 구현 하라.
foldMap, foldRight등 서로를 이용하면 된다.
{% highlight scala %}
trait Foldable[F[_]] {
  def foldRight[A,B](xs: F[A])(z: B)(f: (A,B) => B): B
  def foldLeft[A,B](xs: F[A])(z: B)(f: (B,A) => B): B
  def foldMap[A,B](xs: F[A])(f: A => B)(m: Monoid[B]): B
  def concat[A](xs: F[A])(m: Monoid[A]): A 
}
{% endhighlight %}

### Foldable[List]를 구현하라.
foldRight,foldLeft,foldMap, toList를 구현하라
{% highlight scala %}
class FoldableList extends Foldable[List] {
  override def foldRight[A,B](xs: List[A])(z: B)(f: (A,B) => B): B = ???
  override def foldLeft[A,B](xs: List[A])(z: B)(f: (B,A) =>B): B = ???
  override def foldMap[A,B](xs: List[A])(f: A => B)(m: Monoid[B]): B = ???
  override def toList[A](xs: List[A]): List[A] = ???
}
{% endhighlight %}

### Foldable[IndexedSeq] 구현하라.
//foldRight,foldLeft,foldMap 를 구현하라.
{% highlight scala %}
class FoldableIndexedSeq extends Foldable[IndexedSeq]  {
  override def foldRight[A,B](xs: IndexedSeq[A])(z: B)(f: (A,B) => B): B = ???
  override def foldLeft[A,B](xs: IndexedSeq[A])(z: B)(f: (B,A) => B): B = ???
  override def foldMap[A,B](xs: IndexedSeq[A])(f: A => B)(m: Monoid[B]): B = ???
}
{% endhighlight %}

### Foldable[Stream] 구현하라.
foldRight, foldLeft를 구현하라.
{% highlight scala %}
class FoldableStream extends Foldable[Stream] {
  override def foldRight[A,B](xs: Stream[A])(z: B)(f: (A,B) => B): B = ???
  override def foldLeft[A,B](xs: Stream[A])(z: B)(f: (B,A) => B): B = ???
}
{% endhighlight %}

### Foldable[Tree] 구현하라.
foldRight,foldLeft,foldMap  구현하라.
{% highlight scala %}
sealed trait Tree[+A]
case class Leaf[A](v: A) extends Tree[A]
case class Branch[A](l: Tree[A], r: Tree[A]) extends Tree[A]

class FoldableTree extends Foldable[Tree] {
  override def foldRight[A,B](t: Tree[A])(z: B)(f: (A,B) => B) : B = ???
  override def foldLeft[A,B](t: Tree[A])(z: B)(f: (B,A) => B): B = ???
  override def foldMap[A,B](t: Tree[A])(f: A => B)(m: Monoid[B]): B = ???
}
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture