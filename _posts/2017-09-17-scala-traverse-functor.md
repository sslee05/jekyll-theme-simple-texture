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
def sequenceMap[K, V](xs: Map[K, F[V]]): F[Map[K, V]] =
  xs.foldLeft[F[Map[K, V]]](unit(Map.empty)) {
    case (b, (k, v)) => map2(b, v)((b1, v1) => b1 + (k -> v1))
}
{% endhighlight %}
위의 내용을 보면 구현형태가 비슷하다. 다만 해당 자료구조에 따라 달라진다.  
이들의 공통점을 추상화 하면 traversable functor라는 것을 만날 수 있다.

# traversable Functor의 추상화
{% highlight scala %}
trait Traverse[F[_]] {
  def traverse[G[_]: Applicative, A, B](xs: F[A])(f: A => G[B]): G[F[B]]
  def sequence[G[_]: Applicative, A](xs: F[G[A]]): G[F[A]] = 
    traverse(xs)(a => a)
}
{% endhighlight %}

자료구조와 함수를 받고 자료구조에 담긴 원소들을 함수에 적용해서 결과를 도출하는 면에서 fold 연산과 비슷하지만, 이는 결과가 원래의 자료구조를 보존하는 하지만 foldMap은 구조를 페기하고 Monoid의 이항연산으로 대처된다.(아래의 Foldable의 foldMap code 참조)
{% highlight scala %}
def foldMap[A,B](xs: F[A])(f: A => B)(m: Monoid[B]): B = 
  foldRight(xs)(m.zero)((a,b) => m.op(f(a),b))
{% endhighlight %}



[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
