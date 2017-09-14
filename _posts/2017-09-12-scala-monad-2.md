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

# Monad instance
모나드 typeclass를 가지고 해당하는 자료구조의 Monad를 생성시 Monad가 되기 위한 operation set이 3가지 유형이 있다.  

우선 예로 Option manad를 생성해보자.


{% highlight scala %}
trait Monad[F[_]] {
  def unit[A](a: => A): F[A]
  def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B]
  def map2[A,B,C](ma: F[A])(mb: F[B])(f: (A,B) => C): F[C] = 
    flatMap(ma)(a => map(mb)(b => f(a,b)))
}
{% endhighlight %}





##



[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
