---
layout: post
title: "scala Applicative Functor"
description: "Applicative Functor"
categories: [language]
tags: [scala]
redirect_from:
  - /2017/09/08/
---

> scala Applicative Functor


* Kramdown table of contents
{:toc .toc}

# Applicative Functor
Monad에서 이야기 하면서 box에서 값을 꺼내고, 다른 box에서 사상,값을 꺼내서 적용할 수 있게 하는 구조가 Applicative Functor라 했다.  
Monad가 flatMap으로 대표 된다면, Applicative Functor는 map2라는 것으로 대표된다.  
이들은 각각 장단점이 있으며 그 차이점들을 찾아가 보자.

# Applicative Functor trait
Applicative functor는 다음의 함수 unit 항등함수와 map2의 함수를 가진다.  
또한 map함수를 map2와 unit으로 구현함으로써 Applicative functor 또한 Functor의 하위 구조가 될 수 있음을 알 수 있다.  
따라서 Applicative functor도 Functor의 성질인 구조적 보존의 법칙을 지켜야 한다.  
{% highlight scala %}
trait Applicative[F[_]] extends Functor[F] {
  // 기본수단 
  def unit[A](a: => A): F[A]
  def map2[A,B,C](fa: F[A], fb: f[B])(f: (A,B) => C): F[C]
  
  //파생 function
  def map[A,B](fa: F[A])(f: A => B): F[B] = 
    map2(fa,unit())((a,_) => f(a))
}
{% endhighlight %}




[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
