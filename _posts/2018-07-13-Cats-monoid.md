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
{% endhighlight %}

## operator associativity & identity law
{% highlight scala %}
def associativeLaw[A](a: A, b:A, c:A)(implicit monoid: Monoid[A]): Boolean = 
  monoid.combine(a, monoid.combine(b, c)) ==  monoid.combine(monoid.combine(a,b),c) 
  
def identityLaw[A](a:A)(implicit monoid: Monoid[A]): Boolean = 
  (monoid.combine(a,monoid.empty) == a) && (monoid.combine(monoid.empty,a) == a)
{% endhighlight %}


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
