---
layout: post
title: "scala Traversable Functor"
description: "Traversable Functor"
categories: [language]
tags: [scala]
redirect_from:
  - /2017/09/08/
---

> scala Traversable Functor


* Kramdown table of contents
{:toc .toc}

# 순회가능 형식생성자의 공통 조합기 추상화 하기
List, Map등과 같은 형식생성자들의 자료구조와 함수를 받고 자료구조에 담긴 자료에 함수를 적용하여 원래의 구조를 보존의 기능을 추상화 한 Functor가 Traversable functor이다.  

우선 기존의 traverse, 혹은 sequence 함수들을 보면 다음과 같았다.
{% highlight scala %}
def traverse[F[_],A,B](xs: List[A])(f: A => F[B]): F[List[B]] = 
  xs.foldRight[F[List[A]]](unit(List())((a,b) => map2(f(a),b)((a1,b1) => a1::b1))
def sequence[F[_],A](xs: List[A]): F[List[A]] = 
  xs.foldRight[F[List[A]]](unit(List()))((a,b) => map2(a,b)(a1,b1) => a1 :: b1))
{% endhighlight %}

이를 Map에도 적용해보자.
{% highlight scala %}

{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
