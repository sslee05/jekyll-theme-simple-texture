---
layout: post
title: "scala Monad"
description: "scala Monad"
categories: [scala-function]
tags: [scala,Monad,스칼라,모나드]
redirect_from:
  - /2017/09/11/
---

> scala Monad


* Kramdown table of contents
{:toc .toc}

# 들어가기 앞서
저번 post에서 Monoid와 Functor에 관해 이야기를 했다.  
Functor는 어떤 box(higher-kinded type)에 함수를 적용 할 수 있는 방법을 가진 녀석이라고 했다.  
그리고 Functor에 항등함수(Id)를 적용하면 원래의 Functor와 같으며, 두 함수를 합성한 후 Functor를 적용한 것은 한 함수에 Functor를 적용한 후의 Functor에 나머지 함수를 적용한 것과 같다고 했다.  

scala 를 하면서 모나드, 모나드 무지 많이 사용하기 때문일 것 이다. 이를 사용하면 코드 중복를 피하는 것 뿐아니라 for comprehension 표현으로 가능 함을 제공한다. 이제 Monad와 Monad를 구성하는 함수set들을 살펴 보자.

# Monad
Functor가 어떤 box에 함수적용을 알고 있는 것이라면 Monad는 함수를 적용하고 다시 box에 담는 것이라 할 수 있다. 따라서 결과를 다시 모나드로 보낼 수 있다.  
위의 정의만 봐서는 Monad는 Functor이기도 함을 알 수 있다. 아래에 flatMap과 unit( 함수 이름을 unit보다는 pure로 많이 표현함.)로 map을 구현 할 수 있다. 추후 적용함수도 구현 가능 하다.  

우선 box에 값을 꺼내어 함수를 적용하는 것을 Function 라 했고 표현을 아래와 같이 했다.
{% highlight scala %}
def map[A,B](ma: F[A])(f: A => B): F[B]
{% endhighlight %}
그러면 Monad는 box에서 r값을 꺼내 함수를 적용하고 다시 box에 담는 것이니 아래와 같이 표현하면 될 듯 하다.
{% highlight scala %}
def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B]
{% endhighlight %}
f를 보면 box의 값에서  f를 적용하면 결과로 다시 box인 F[B]로 반환 한다.  

Monad를 typeclass로 나타내면 다음과 같다.  
{% highlight scala %}
trait Monad[F[_]] {
  // 항등함수 unit 일반적으로  pure라고 이름을 많이 함
  def unit[A](a: => A): F[A] 
  //bind 함수(하스켈에서는 >>= 라 표현한다고 함.)
  def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B]
}
{% endhighlight %}
Monoid, Functor도 법칙이 있 듯 물론 Monad도 법칙이 있다.

# Monad Law
<span style="color:red">1.*Left 항등법칙*</span>
<span style="color:red">2.*Right 항등법칙*</span>
<span style="color:red">3.*bind함수에 대한  결합법칙*</span>

## Left 항등법칙
{% highlight scala %}
unit(x) flatMap(f) == f(x)

scala> def f:Int => Option[Int] = a => Some(a * 2)
f: Int => Option[Int]

scala> def unit[Int](a: Int): Option[Int] = Some(a)
unit: [Int](a: Int)Option[Int]

scala> def flatMap[A,B](ma: Option[A])(f: A => Option[B]): Option[B] = ma flatMap f
flatMap: [A, B](ma: Option[A])(f: A => Option[B])Option[B]

scala> val x = 1
x: Int = 1

scala> flatMap(unit(x))(f) == f(x)
res4: Boolean = true

{% endhighlight %}


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
