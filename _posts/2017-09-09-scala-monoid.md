---
layout: post
title: "scala monoid"
description: "scala monoid"
categories: [language]
tags: [scala]
redirect_from:
  - /2017/09/08/
---

> scala monoid.
>
> 이 내용은 수학책과 "Functional Programming in scala" 책을 공부하며 정리한 것 임.  
> (잘 못 이해한 것 또는 틀린 내용이 있을 수 있음.)

* Kramdown table of contents
{:toc .toc}

# 수학적 기본 지식

## 대수란
정수나 유리수와 같이 우리에게 익숙한 수들의 기본 __성질__ 들을 추상화, 일반화하는 것을 포함하는 
아이디어와 기능으로 구성된 수학적 세계 중에 하나.

## 대수학 
정수가 가지고 있는 __성질__ 의 일반화에 대한 연구이다.

## 군
__결합법칙__, __항등법칙__ , __교환법칙__ 이 성립하는 성질을 임의의 집합으로 일반화 한 것이 군이다.  
__✶__ 를 하나의 집합 G에 대한 __이항 연산__ 이라 한다면( 임이의 x,y ∈ G => x ✶ y ∈ G )
다음의 성질을 만족하면 __G__ 또는 __(G,✶)__ 를 군이라 한다.  
   l-1  ✶에 대한 결합법칙 ) x,y,z ∈ G => x ✶ (y ✶ z) = (x ✶ y) ✶ z  
   l-2 항등원과 ✶에 대한 항등법칙) 모든 x ∈ G 에 대하여 e ✶ x = x ✶ e = x 을 만족하는 e ∈ G 가 존재한다.  
   l-3 x는 역원를 갖는다) 각각의 x ∈ G 에 대하 x^-1 ∈ G 가 존재하며 x ✶ x^-1 = x^-1 ✶ x = e   
추가로 다음을 만족하면 가환군(아벨군)이라 한다.  
   l-4) x,y ∈ G => x ✶ y = y ✶ x   : ✶ 에 대한 교환법칙

## 사상
대상의 관계에 대한 mapping, 혹은 function ( 임이의 군 __=>__ 임이의 군 )

## 모노이드(Monoid)
✶를 하나의 집합 G에 대한 이항 연산이라 한다면( 임이의 x,y ∈ G => x ✶ y ∈ G ) l-1,l-2,l-3의 성질을 가지는 군을 monoid라 한다.
{% highlight scala %}
trait Monoid[A] {
	def op(a: A, b: A): A = ??? //( 결합법칙 이 성립하게 구현)
	def zero: A = ??? //(항등원)
}
{% endhighlight %}

operation에 인자들은 A 유형이다.  
a,b ∈ A 이고 zero(항등원) ∈ A 이며 (구현은 안되어 있지만, 결합법칙이 성립하는 연산) op의 이항연산이 있다.  
이때 위의 군의 정의에 의해 A 또는 Monoid[A]의 인스턴스를 monoid 라 한다.  


# scala로 Monoid를 모델링 해보자.
이제 Monoid가 무었인지 알았으니 Monoid 에를 작성하며 연습해보자.

ex-01) String monoid를 만들어 보자.
{% highlight scala %}
def stringMonoid: Monoid[String] = new Monoid[String] {
  override def op(a: String, b: String): String = a + b
  override def zero: String = ""
}
{% endhighlight %}

ex-02) Int 덧셈 monoid를 선언 만들어 보자.
{% highlight scala %}
def intMonoid: Monoid[Int] = new Monoid[Int] {
  override def op(a: Int, b: Int): Int = a + b
  override def zero: Int = 0
}
{% endhighlight %}

ex-03)Int 곱셈에 대한 Monoid를 만들어 보자.
{% highlight scala %}
def intProductMonoid: Monoid[Int] = new Monoid[Int] {
  override def op(a: Int, b: Int): Int = a * b
  override def zero: Int = 1
}
{% endhighlight %}

ex-04) Option 에 대한 Monoid를 만들어 보자.
{% highlight scala %}
def optionMonoid[A]: Monoid[Option[A]] = new Monoid[Option[A]] {
  override def op(o1: Option[A], o2: Option[A]): Option[A] = o1.orElse(o2)
  override def zero: Option[A] = None
}
{% endhighlight %}
위의 optionMonoid.op도 결합법칙이 성립된다.
{% highlight scala %}
val o1 = Some("a")
val o2 = Some("b")
val o3 = Some("c")
//Option[String] 에 대한 결합법칙 
val rs01 = optionMonoid.op(o1,optionMonoid.op(o2,o3))
val rs02 = optionMonoid.op(optionMonoid.op(o1,o2),o3)
println("rs01 :"+ rs01+ " rs02:"+rs02)
{% endhighlight %}
o1,o2,o3 중 어느 것에 None를 넣어도 성립된다.  


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture