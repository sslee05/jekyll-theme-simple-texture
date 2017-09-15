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
모나드의 bind 함수인 flatMap의 결합법칙 정의는 다음과 같다.  
{% highlight scala %}
//결합법칙 
x.flatMap(f).flatMap(g) == x.flatMap(a => f(a).flatMap(g))

//Some(v) 를 x 에 대입
Some(v).flatMap(f).flatMap(g) == Some(v).flatMap(a => f(a).flatMap(g))
f(v).flatMap(g) == (a => f(a).flatMap(g))(v)
f(v).flatMap(g) == f(v).flatMap(g)
{% endhighlight %}

## flatMap의 항등법칙
### left 항등법칙
{% highlight scala %}
unit(x) flatMap(f) == f(x)
{% endhighlight %}
### right 항등법칙
{% highlight scala %}
ma flatMap(unit) == ma 
{% endhighlight %} 

# Identity Monad
이는 그냥 간단한 wrapper에 해당하는 역할을 한다.  
{% highlight scala %}
case class Id[A](value: A) {
  def unit(a: A): Id[A] = Id(a)
  def map[B](f: A => B): Id[B] = Id(f(value))
  def flatMap[B](f: A => Id[B]): Id[B] = f(value)
}
{% endhighlight %}

# State Monad
State Monad bind operator는 결국 상태전이 역할을 한다.  
자 우선 위에 해왔던 Monad 를 보자.
{% highlight scala %}
trait Monad[F[_]] {
 .....
}
{% endhighlight %}
그리고 State 를 보자
{% highlight scala %}
case class State[S,A](run: S => (S,A))
{% endhighlight %}
State는 형식을 2개 받고 있고 Monad[State[S,A]] 라고 할 수 없다.  
이 문제를 해결하기 위해 다음 code를 보자  
{% highlight scala %}
def f1(a: Int)(f: Int => Double): Double = f(a)
val f2 = f1(3) _
println(f2(a => a * a))
println(f2(a => a.toDouble / (a+1)))
{% endhighlight %}
f2는 3을 인자로 고정된 함수 instance가 되었다.  부분적용 함수를 한 것 이다.  
이처럼 State[S, _ ] 형태, 즉 State[S, _ ]가 고정된 가변유형 하나를 포함하는 또 다른 하나의 형태구조라 생각 하고 접근하면 된다.  
결국 State는 run 함수를 감싼 box라 생각하면 된다. 그리고 run를 통해 box안의 값, 함수 S => (S,A)를 꺼내면 된다.  
이 또 하나의 형태 State[S,_]는 S 유형이 고정이며 값에 따라 상태(S)가 전이가 된다.  
이때 다음과 같이 inline 표기를 scala는 지원한다.  
{% highlight scala %}
def stateMonad[S] = new MonadSet01[({type f[X] = State[S,X]})#f] {
 .....
}
{% endhighlight %}
이제 stateMonad를 구현 해보자.
{% highlight scala %}
def stateMonad[S] = new MonadSet01[({type f[X] = State[S,X]})#f] {
  def unit[A](a: => A): State[S,A] = State(s => (a,s))
  def flatMap[A,B](ms: State[S,A])(f: A => State[S,B]): State[S,B] = 
	ms flatMap f
}

//State code 부분
case class State[S,+A](run:S => (A,S)) {
  def flatMap[B](f: A => State[S,B]):State[S,B] = State(s => {
    val (a,s1) = run(s)
    f(a).run(s1)
  })
  
  def unit[S,A](a:A):State[S,A] = State(s => (a,s))
  def map[B](f:A => B):State[S,B] = 
    flatMap(a => unit(f(a)))
  def map2[B,C](s:State[S,B])(f:(A,B) => C):State[S,C] = 
    flatMap(a => s.map(b => f(a,b)))
}
{% endhighlight %}
위의 코드를 State를 생각하지 말고 안에 있는 function 의 변환를 생각 해보면 bind함수(flatMap)이 한 일을 좀더 쉽게 알 수 있다.  
(s => (a,s1)) bind (a => (s1 => (b,s2)))


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
