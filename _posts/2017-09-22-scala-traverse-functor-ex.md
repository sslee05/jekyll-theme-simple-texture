---
layout: post
title: "scala Traversable Functor exercise"
description: "Traversable Functor exercise"
categories: [language]
tags: [scala-exercise]
redirect_from:
  - /2017/09/08/
---

> scala Traversable Functor


* Kramdown table of contents
{:toc .toc}

# Traversable Functor
### Functor trait를 만들고 Functor를 상속하는 Applicative Functor, Traversable Functor triat를 만들어라.
{% highlight scala %}
trait Functor[F[_]] {
  def map[A, B](a: F[A])(f: A => B): F[B]
}
trait Applicative[F[_]] extends Functor[F] {
  def unit[A](a: => A): F[A]
  def apply[A, B](fab: F[A => B])(ma: F[A]): F[B]
  
  def map2[A, B, C](ma: F[A], mb: F[B])(f: (A, B) => C): F[C]
}
trait Traverse[F[_]] extends Functor[F]{
  def traverse[G[_]: Applicative, A, B](xs: F[A])(f: A => G[B]): G[F[B]]
  def sequence[G[_]: Applicative, A](xs: F[G[A]]): G[F[A]] =
      traverse(xs)(a => a)
}
{% endhighlight %}

### Applicative Functor trait 에 apply method를 구현하라.
{% highlight scala %}
def apply[A, B](fab: F[A => B])(ma: F[A]): F[B] = ???}
{% endhighlight %}

### Applicative Functor trait 에 apply를 이용하여 map2를 구현하라.
{% highlight scala %}
def map2[A, B, C](ma: F[A], mb: F[B])(f: (A, B) => C): F[C] = ???
{% endhighlight %}

### Applicative Functor trait 에 apply를 이용하여 map를 구현하라.
{% highlight scala %}
def map[A, B](ma: F[A])(f: A => B): F[B] = ???
{% endhighlight %}

### Applicative Functor trait 에 sequenceList를 구현하라..
{% highlight scala %}
def sequenceList[A](xs: List[F[A]]): F[List[A]] = ???
{% endhighlight %}

### Applicative Functor trait 에 sequenceMap를 구현하라.
{% highlight scala %}
def sequenceMap[K, V](xs: Map[K, F[V]]): F[Map[K, V]] = ???
{% endhighlight %}

### Traverse object를 만들고 그 곳에 listTraverse instance를 만들어라.
### Traverse object에 OptionTraverse instance를 만들어라.
### Traverse object에 TreeTraverse instance를 만들어라.
{% highlight scala %}
object Traverse {
  case class Tree[+A](head: A, tail: List[Tree[A]])
}
{% endhighlight %}

### 다음의 Monad 코드를 채워라
{% highlight scala %}
trait Monad[F[_]] extends Applicative[F] {
  def unit[A](ma: => A): F[A]
  def flatMap[A, B](ma: F[A])(f: A => F[B]): F[B]

  override def map[A, B](m: F[A])(f: A => B): F[B] = ???
  override def map2[A, B, C](ma: F[A], mb: F[B])(f: (A, B) => C): F[C] = ???
  def join[A](mma: F[F[A]]): F[A] = ???
}
{% endhighlight %}

### trait Traverse 에 Functor의 map함수를 구현하라.
{% highlight scala %}
type Id[A] = A

val idMonad = new Monad[Id] {
  def unit[A](a: => A) = a
  override def flatMap[A, B](ma: Id[A])(f: A => Id[B]): Id[B] = f(ma)
}

def map[A, B](ma: F[A])(f: A => B): F[B] = ???

{% endhighlight %}

### trait Traverse 에 Foldable를 상속하고 foldMap를 의 override 하라.
{% highlight scala %}
type Const[M, B] = M
implicit def monoidApplicative[M](m: Monoid[M]) = new Applicative[({ type f[x] = Const[M, x] })#f] {
  def unit[A](a: => A): M = m.zero
  override def map2[A, B, C](m1: M, m2: M)(f: (A, B) => C): M = m.op(m1, m2)
}

override def foldMap[A, M](xs: F[A])(f: A => M)(mb: Monoid[M]): M = ???
{% endhighlight %}

### trait Traverse 에 stateMonad instance를 반환하는 함수를 만들어라.
{% highlight scala %}
import basic.state.State
import basic.state.State._
def stateMonad[S] = new Monad[({ type StateS[A] = State[S, A] })#StateS] {
  def unit[A](a: => A): State[S, A] = State.unit(a)
  override def flatMap[A, B](s: State[S, A])(f: A => State[S, B]): State[S, B] = ???
}
{% endhighlight %}

### trait Traverse 에 State 동작과 traverse 를 이용한 내부상태를 유지하면서 자료구조를 훑는 다음의 함수를 작성하라.
{% highlight scala %}
import basic.state.State
import basic.state.State._
def traverseS[S, A, B](fa: F[A])(f: A => State[S, B]): State[S, F[B]] = ???
}
{% endhighlight %}

### trait Traverse 에 traverseS 함수를 이용해서 0부터 시작하여 1씩증가하는 내부상태 를 구하는 다음의 함수를 구하라.
{% highlight scala %}
def zipWithIndex_[A](ta: F[A]): F[(A, Int)] = ???
{% endhighlight %}

### trait Traverse 에  Traverse Functor를 목록으로 반한하는 다음의 함수를 구하라.
{% highlight scala %}
def toList_[A](ta: F[A]): List[A] = ???
{% endhighlight %}

### trait Traverse 에 상태순회 Traversable Functor를 추상화 하라.
State 를 이용한 순회는  
현재상태을 얻고 -> 다음상태를 계산하고 -> 그것을 현재상태로 갱신하고 -> 어떤 값을 산출(yield)  
하는 패턴으로 일반화 가능하다.
{% highlight scala %}
def mapAccum[S, A, B](fa: F[A], s: S)(f: (A, S) => (B, S)): (F[B], S) = ???
{% endhighlight %}

### trait Traverse 에 mapAccum으로 zipWithIndex를 다시 구현하라.
{% highlight scala %}
def zipWithIndex[A](ta: F[A]): F[(A, Int)] = ???
{% endhighlight %}

### trait Traverse 에 mapAccum으로 toList를 다시 구현하라.
{% highlight scala %}
def toList[A](fa: F[A]): List[A] = ???
{% endhighlight %}

### Applicative trait 에 다른 Applicative Functor를 곱하는 product 함수를 구현하라.
{% highlight scala %}
def product[G[_]](G: Applicative[G]): Applicative[({type f[x] = (F[x],G[x])})#f] = ???
{% endhighlight %}

### trait Traverse 에 traversable Functor의 융합하는 다음을 구현하라.
applicative functor를 이용하여 두순외의 유합을 산출하는 다음의 함수를 작성하라.
두 함수 f 와 g가 주어졌을 때 이함수는 fa를 한 번만 순회해야 한다.
Applicative 에 Applicative functor를 곱하는 product function 이 필요한다.
{% highlight scala %}
def fuse[G[_], H[_], A, B](fa: F[A])(f: A => G[B], g: A => H[B])(implicit G: Applicative[G], H: Applicative[H]): (G[F[B]], H[F[B]]) = ???
{% endhighlight %}

### trait Traverse 에 traverse functor를 합성하는 다음의 함수를 구현하라.
{% highlight scala %}
def compose[G[_]](implicit G: Traverse[G]): Traverse[({type f[x] = F[G[x]]})#f] = ???
{% endhighlight %}

### Monad object를 만들고 그 곳에 monad합성을 하는 다음을 구현하라.
{% highlight scala %}
def composeM[F[_],G[_]](implicit F: Monad[F], G: Monad[G], T: Traverse[G]): Monad[({type f[x] = F[G[x]]})#f] = ???
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
