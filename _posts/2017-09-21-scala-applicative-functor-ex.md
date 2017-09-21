---
layout: post
title: "scala Applicative Functor exercise"
description: "Applicative Functor exercise"
categories: [language]
tags: [scala-exercise]
redirect_from:
  - /2017/09/08/
---

> scala Applicative Functor


* Kramdown table of contents
{:toc .toc}

#Applicative Functor
### unit,map2를 기본조합기로하는 Applicative Functor trait를 작성하라.
{% highlight scala %}
trait Functor2[F[_]] {
  def map[A,B](fa: F[A])(f: A => B): F[B]
}

trait Applicative[F[_]] extends Functor2[F] {
  def unit[A](a: => A): F[A]
  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C]
}
{% endhighlight %}

### applicative functor trait에 map함수를 unit과 map2로 구현하라.
{% highlight scala %}
def map[A, B](m: F[A])(f: A => B): F[B] = ???
{% endhighlight %}

### Applicative trait에 list 용 traverse  함수를 구현하라.
{% highlight scala %}
def traverse[A, B](xs: List[A])(f: A => F[B]): F[List[B]] = ???
{% endhighlight %}

### Applicative trait에 list용 sequence 함수를 구현하라.
{% highlight scala %}
def sequenceByTraverse[A](xs: List[F[A]]): F[List[A]] = ???
{% endhighlight %}

### Applicative trait에 map2 와 unit 으로 replicateM 함수를 구현하라.
{% highlight scala %}
def replicateM[A](n: Int, m: F[A]): F[List[A]] = ???
def replicate[A](n: Int, m: F[A]): F[List[A]] = ???
{% endhighlight %}


### Applicative trait에  map2로 product 함수를 구현하라.
{% highlight scala %}
def product[A, B](ma: F[A], mb: F[B]): F[(A, B)] = ???
{% endhighlight %}

### Applicative trait에 쌍들의 묶음을 바꾸는 다음의 함수를 구현하라.
{% highlight scala %}
def assoc[A,B,C](p: (A,(B,C))): ((A,B),C) = ???
{% endhighlight %}

### Applicative trait에 각각의 Input => Output 2개를 받아 Input 쌍을 받고 Outout 쌍을 받는 함수를 구현하라.
{% highlight scala %}
def productF[I1,I2,O1,O2](f: I1 => O1, g: I2 => O2): (I1,I2) => (O1,O2) = ???
{% endhighlight %}

### Applicative trait에 Applicative Functor의 합성을 구현하라.
{% highlight scala %}
def compose[G[_]](G: Applicative[G]): Applicative[({type f[X] = F[G[X]]})#f] = ???
{% endhighlight %}

### ApplicativeV2 trait를 만들어 그 곳에 기본수단을 unit과 map2대신 unit과 apply 로 구성하고 map 과 map2를 unit과 apply로 구현하라.
{% highlight scala %}
trait ApplicativeV2[F[_]] extends Functor2[F] {
  def unit[A](a: => A): F[A]
  def apply[A, B](fab: F[A => B])(fa: F[A]): F[B]

  def map[A, B](m: F[A])(f: A => B): F[B] = ???
  def map2[A, B, C](ma: F[A], mb: F[B])(f: (A, B) => C): F[C] = ???
}
{% endhighlight %}

### ApplicativeV2 trait에 apply 로 map3,map4를 구혀나라.
{% highlight scala %}
def map3[A, B, C, D](ma: F[A], mb: F[B], mc: F[C])(f: (A, B, C) => D): F[D] = ???
def map4[A, B, C, D, E](ma: F[A], mb: F[B], mc: F[C], md: F[D])(f: (A, B, C, D) => E): F[E] = ???
{% endhighlight %}

### Applicative3 trait를 만들어 그 곳에 기본수단을 unit,map,map2로 하고 이를 이용하여 apply를 구현하라.
{% highlight scala %}
trait ApplicativeV3[F[_]] extends Functor2[F] {
  def unit[A](a: => A): F[A]
  def map[A, B](a: F[A])(f: A => B): F[B]
  def map2[A, B, C](ma: F[A], mb: F[B])(f: (A, B) => C): F[C]
  
  def apply[A, B](fo: F[A => B])(m: F[A]): F[B] = ???
}
{% endhighlight %}

### Applicative trait에 apply를 map2와 map으로 구현하라.
{% highlight scala %}
def apply[A, B](fo: F[A => B])(m: F[A]): F[B] = ???
{% endhighlight %}


### Applicative object를 만들고 그곳에 다음의 함수를 구현하라.
{% highlight scala %}
val optionAF = new Applicative[Option] = ???
{% endhighlight %}

### Applicative object를 만들고 그곳에 다음의 함수를 구현하라.
{% highlight scala %}
def eitherMonad[E]: basic.monad.MonadStudy.Monad[({ type f[X] = Either[E, X] })#f] = ???
{% endhighlight %}

### Applicative object에 다음의 validation을 위한 Applicative functor를 구현하라.
{% highlight scala %}
trait Validation[+E, +A]
case class Failure[E](head: E, tail: Vector[E] = Vector()) extends Validation[E, Nothing]
case class Success[A](a: A) extends Validation[Nothing, A]
  def validator[E] = new Applicative[({ type f[X] = Validation[E, X] })#f] {
  def unit[A](a: => A): Validation[E, A] = ???
  def map2[A, B, C](ma: Validation[E, A], mb: Validation[E, B])(f: (A, B) => C): Validation[E, C] = ???
}
{% endhighlight %}


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
