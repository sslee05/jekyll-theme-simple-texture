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
(함수와 메소드는 다르지만 설명을 위해 구분을 따로 하지 않고 함수라 표현 했다.)

# Monad
Functor가 어떤 box에 함수적용을 알고 있는 것이라면 Monad는 함수를 적용하고 다시 box에 담는 것이라 할 수 있다. 따라서 결과를 다시 모나드로 보낼 수 있다.  
위의 정의만 봐서는 Monad는 Functor이기도 함을 알 수 있다. 아래에 flatMap과 unit( 함수 이름을 unit보다는 pure로 많이 표현함.)로 map을 구현 할 수 있다. 추후 적용함수도 구현 가능 하다.  

우선 box에 값을 꺼내어 함수를 적용하는 것을 Function 라 했고 표현을 아래와 같이 했다.
{% highlight scala %}
def map[A,B](ma: F[A])(f: A => B): F[B]
{% endhighlight %}
그러면 Monad는 box에서 값을 꺼내 함수를 적용하고 다시 box에 담는 것이니 아래와 같이 표현하면 될 듯 하다.
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

scala> def unit[A](a: A): Option[A] = Some(a)
unit: [A](a: A)Option[A]

scala> def flatMap[A,B](ma: Option[A])(f: A => Option[B]): Option[B] = ma flatMap f
flatMap: [A, B](ma: Option[A])(f: A => Option[B])Option[B]

scala> def f:Int => Option[Int] = a => Some(a * 2)
f: Int => Option[Int]

scala> val x = 2
x: Int = 2

scala> flatMap(unit(x))(f) == f(x)
res0: Boolean = true
{% endhighlight %}

Monad 의 적용 결과는 모나드의 입력으로 그결과는 또다른 입력으로 Monad chain이 가능하다.  
Monad는 행간의 무었이 일어나느지 명시하는 것이 아닌, 단지 어떤 일이 일어나더라도 그것이 결합법칙과 항등법칙을 만족함을 명시할 뿐이다.

## Right 항등법칙
{% highlight scala %}
f(x) flatMap (unit) == f(x)

scala> def unit[A](a: A): Option[A] = Some(a)
unit: [A](a: A)Option[A]

scala> def flatMap[A,B](ma: Option[A])(f: A => Option[B]): Option[B] = ma flatMap f
flatMap: [A, B](ma: Option[A])(f: A => Option[B])Option[B]

scala> def f:Int => Option[Int] = a => Some(a * 2)
f: Int => Option[Int]

scala> val x = 2
x: Int = 2

scala> flatMap(f(x))(unit) == f(x)
res2: Boolean = true
{% endhighlight %}

## flatMap 연산에 대한 결합법칙
{% highlight scala %}
F flatMap f flatMap g = F flatMap(a => f(x) flatMap g)

scala> def unit[A](a: A): Option[A] = Some(a)
unit: [A](a: A)Option[A]

scala> def flatMap[A,B](ma: Option[A])(f: A => Option[B]): Option[B] = ma flatMap f
flatMap: [A, B](ma: Option[A])(f: A => Option[B])Option[B]

scala> val f: Int => Option[Int] = a => Some(a + 2)
f: Int => Option[Int] = $$Lambda$1063/1354001956@1c30cb85

scala> val g: Int => Option[Int] = a => Some(a * 2)
g: Int => Option[Int] = $$Lambda$1064/1816978819@755b5f30

scala> val ma = Some(1)
ma: Some[Int] = Some(1)

scala> val f: Int => Option[Int] = a => Some(1)
f: Int => Option[Int] = $$Lambda$1051/994712181@65bad087

scala> (ma flatMap f flatMap g) == (ma flatMap(a => f(a) flatMap g))
res5: Boolean = true
{% endhighlight %}


# Monad 의 다른 조합 유형
## compose
어떤 higher-kinded type이 위의 Monad법칙을 만족시키고 다음과 같은 bind operator를 가진다면 Monad라 할 수 있다.  
flatMap[B](f: A => F[B]): F[B]  

그럼 또 하나의 아래와 같은 signatuer를 보자 
{% highlight scala %}
def compose[A,B,C](f: A => F[A])(g:B => F[C]):A => F[C] 
{% endhighlight %}
위와 같은 함수로 flatMap를 구현 할 수 있을까? 아래 코드를 보자  

{% highlight scala %}
def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B] = 
 compose( (_:Unit) => ma, f)(())
{% endhighlight %}
가능하다. 그럼 Monad의 구성 set은 unit 과 compose 구성으로 가능한 또 하나의 구성이 생긴다.  

## flatten
위의 Monad 정의에서 Monad는 Functor이기도 하다라고 했다. 그렇다면 Monad의 구성 set으로 map함수를 만들 수 있어야 한다. map함수는 다음과 같았다.
{% highlight scala %}
def map[A,B](ma: F[A])(f: A => B): F[B]
{% endhighlight %}
이를 unit과 flatMap 으로 아래와 같이 구현 할 수 있다.  
{% highlight scala %}
def map[A,B](ma: F[A])(f: A => B): F[B] = 
  flatMap(ma)(a => unit(f(a)))
{% endhighlight %}

이제 Monad는 Functor 임을 알 수 있다.  
다음으로 flatten이라는 다음과 같은 함수가 있다고 하자. 
{% highlight scala %}
def flatten[A](xs: F[F[A]]): F[A]
{% endhighlight %}
자 그럼 map과 flatten으로...  
맞다 flatMap을 구현 할 수 있다.  
{% highlight scala %}
def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B] = 
  flatten(map(ma)(f))
{% endhighlight %}
이로써 Monad의 또 하나의 구성이 되었다.  

* unit, flatMap  
* unit, componse  
* unit, flatten

# Monad의 합성? (Composed Monad ?) 
Monad의 합성을 해볼까?  즉 Monad F 와 Monad G의 합성으로 된  F[G[\_]]인 Monad를 해볼까?  
될까? 안될까?  
무작정 시도 해보자  

- 여기에 나오는 context bound 및 람다type(#f는 f[x]에 대한 type project, f[x]는 type F[G[x]] 를 나타 낸다.)은 나중에..  

  

{% highlight scala %}
def composeMonad[F[_],G[_])(implicit F[_]: Monad[F], G[_]: Monad[G]): Monad[({type f[x] = F[G[x]]})#f] = new Monad[({type f[x] = F[G[x]]})#f] {
  def unit[A](a: => A): F[G[A]] = F.unit(G.unit(a))
  def flatMap[A,B](mga: F[G[A]])(f: A => F[G[B]]): F[G[B]] = ???
    //F.flatMap(mga)(ga => G.map(ga)(f)) ??? F[G[F[G[B]]]] ?????
}
{% endhighlight %}
위에  G.map(ga)(f)의 결과는 G[F[G[B]]]이다....  
이를 위해서는 sequence\[A\](ma: F\[G\[A\]\]):G\[F\[A\]\]가 필요하다.. 이는 추후 Traverse 순회적용함수자에서 보고 여기서 Monad post를 마친다.  













[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
