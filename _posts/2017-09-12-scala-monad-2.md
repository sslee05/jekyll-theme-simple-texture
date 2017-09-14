---
layout: post
title: "scala Monad (2)"
description: "scala Monad (2)"
categories: [language]
tags: [scala]
redirect_from:
  - /2017/09/08/
---

> scala Monad


* Kramdown table of contents
{:toc .toc}

# Monad typeclass 만들기
Monoid처럼 항등함수가 있어야 하고  
Monad는 box에서 값을 꺼내고 box에서 값 또는 함수를 꺼내서 operation 할 수 있게 하는 bind함수가 필요하다.  
{% highlight scala %}
trait Monad[F[_]] {
  def unit[A](a: => A): F[A]
  def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B]
  
  def map2[A,B,C](ma: F[A])(mb: F[B])(f: (A,B) => C): F[C] =
    flatMap(ma)(a => map(mb)(b => f(a,b)))
}
{% endhighlight %}

# Monad 조합기만들기 
모나드 typeclass를 가지고 해당하는 자료구조의 Monad를 생성시 Monad가 되기 위한 operation set이 3가지 유형이 있다.  

## flatMap
unit,flatMap 직접 구현하기.
{% highlight scala %}
val optionM = new Monad[Option] {
  def unit[A](a: => A) : Option[A] = Some(a)
  def flatMap[A,B](o: Option[A])(f: A => Option[B]): Option[B] = 
	o flatMap(a => f(a))
}
{% endhighlight %}

## unit,compose
compose라는 합성함수를 통해 flatMap를 구현 할 수 있다.  
{% highlight scala %}
def compose[A,B,C](fa: A => F[B], fb: B => F[C]): A => F[C]
{% endhighlight %}
그럼 Monad trait는 다음과 같아 진다.
{% highlight scala %}
trait Monad[F[_]]  {
  def unit[A](a: => A): F[A]
  def compose[A,B,C](fa: A => F[B], fb: B => F[C]): A => F[C] 
  
  //unit과 compose만 있으면 아래는 자동 구현 됨.
  def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B] = 
    compose((_: Unit) => ma, f)(())
  def map[A,B](m: F[A])(f: A => B): F[B] = 
    flatMap(m)(a => unit(f(a)))
}
{% endhighlight %}

이를 통해 monad를 생성할때 compose와 unit 함수가 있으면 monad 조건이 된다.  
{% highlight scala %}
val moByCompose = new Monad[Option] {
  def unit[A](a: => A) : Option[A] = Some(a)
  def compose[A,B,C](o1: A => Option[B], o2: B => Option[C]): A => Option[C]  = a => o1(a) flatMap(a1 => o2(a1))
}
{% endhighlight %}

## unit,map,join
join이라는 함수를 통해 flatMap은 map,join 으로 만들 수 있다.  
join 의 함수 sygnature는 다음과 같다.
{% highlight scala %}
def join[A](mma: F[F[A]]): F[A]
{% endhighlight %}
이를 통행 monad를 생성할때 join,unit,map 함수가 있으면 monad 조건이 된다.  
{% highlight scala %}
trait Monad[F[_]] {
  def unit[A](a: => A): F[A]
  def map[A,B](ma: F[A])(f: A => B): F[B]
  def join[A](mma: F[F[A]]): F[A] 

  //join map만 있으면 아래는 자동 구현 됨.
  def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B] = 
    join(map(ma)(a => f(a)))
{% endhighlight %}
이를 통해 monad를 생성할때 join 과 map 함수가 있으면 monad 조건이 된다.  

## manad 조합기 set 정리
monad는 monadic 조합기들의 최소 집합 중 하나를 결합버칙과 항등법칙을 만족하도록 구현한 것이다.  
아래는 조합기들의 최소 집합 3가지 set
- unit,flatMap 직접구현 set
- unit,compose 구현 set
- unit,map,join 구현 set

# Monad Law 증명 
## flatMap(bind 함수) 결합법칙
모나드의 결합법칙 정의는 다음과 같다.
{% highlight scala %}
x.flatMap(f).flatMap(b) = x.flatMap(a => f(a).flatMap(g))
{% endhighlight %}


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
