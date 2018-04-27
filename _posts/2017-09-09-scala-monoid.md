---
layout: post
title: "scala Monoid"
description: "scala Monoid"
categories: [monad와 친구들]
tags: [scala,스칼라,Monoid, 모노이드]
redirect_from:
  - /2017/09/08/
---

> scala Monoid.
>


* Kramdown table of contents
{:toc .toc}

# 수학적 기본 지식

## 대수란
정수나 유리수와 같이 우리에게 익숙한 수들의 기본 __성질__ 들을 추상화, 일반화하는 것을 포함하는 
아이디어와 기능으로 구성된 수학적 세계 중에 하나.

## 대수학 
수가 가지고 있는 __성질__ 의 일반화에 대한 연구이다.

## 군
__결합법칙__, __항등법칙__ , __교환법칙__ 이 성립하는 성질을 임의의 집합으로 일반화 한 것이 군이다.  
__✶__ 를 하나의 집합 G에 대한 __이항 연산__ 이라 한다면( 임이의 x,y ∈ G => x ✶ y ∈ G )
다음의 성질을 만족하면  __(G,✶)__ 를 군이라 한다.  
### l-1  ✶ 연산에 대한 결합법칙  
x,y,z ∈ G => x ✶ (y ✶ z) = (x ✶ y) ✶ z  
### l-2 항등원과 ✶에 대한 항등법칙  
모든원소 각각에 x ∈ G 에 대하여 e ✶ x = x ✶ e = x 을 만족하는 e ∈ G 가 존재한다.  
### l-3 x는 역원를 갖는다  
각각의 x(모든 원소) ∈ G 에 대하 x^-1 ∈ G 가 존재하며 x ✶ x^-1 = x^-1 ✶ x = e   
추가로 다음을 만족하면 가환군(아벨군)이라 한다.  
### l-4) x,y ∈ G => x ✶ y = y ✶ x     (✶ 에 대한 교환법칙)

## 사상
대상의 관계에 대한 mapping, 혹은 function ( 임이의 군 __=>__ 임이의 군 )

## 모노이드(Monoid)
✶를 하나의 집합 G에 대한 이항 연산이라 한다면( 임이의 x,y ∈ G => x ✶ y ∈ G ) 이고 l-1,l-2 성질을 가지는 군을 monoid라 한다.


# scala로 Monoid를 모델링 해보자.
아래는 monoid 의 typeclass 이다.  
(A,op)를 나타낸다.
{% highlight scala %}
trait Monoid[A] {
  def op(a: A, b: A): A = ??? //( 결합법칙 이 성립하게 구현)
  def zero: A = ??? //(항등원)
}
{% endhighlight %}

operation에 인자들은 A 유형(A집합의 원소)이다.  
a,b ∈ A 이고 zero(항등원) ∈ A 이며 (구현은 안되어 있지만, 결합법칙이 성립하는 연산) op의 이항연산이 있다.  
이때 위의 군의 정의에 의해 Monoid[A]의 인스턴스를 monoid 라 한다.  

이제 Monoid가 무었인지 알았으니 Monoid 에를 작성하며 연습해보자.

ex-01) String monoid를 만들어 보자.
아래는 (String,+) 군에 대한
{% highlight scala %}
def stringMonoid: Monoid[String] = new Monoid[String] {
  override def op(a: String, b: String): String = a + b
  override def zero: String = ""
}
{% endhighlight %}

ex-02) Int 덧셈 monoid를 선언 만들어 보자.
아래는 (Int,+) 군에 대한 
{% highlight scala %}
def intMonoid: Monoid[Int] = new Monoid[Int] {
  override def op(a: Int, b: Int): Int = a + b
  override def zero: Int = 0
}
{% endhighlight %}

ex-03)Int 곱셈에 대한 Monoid를 만들어 보자.
아래는 (Int,*) 군에 대한 
{% highlight scala %}
def intProductMonoid: Monoid[Int] = new Monoid[Int] {
  override def op(a: Int, b: Int): Int = a * b
  override def zero: Int = 1
}
{% endhighlight %}

ex-04) Option 에 대한 Monoid를 만들어 보자.
(Option,orElse) 군에 대한
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

ex-05) 자기함수: 인수의 형식과 반환값의 형식이 동일한 함수  
자기함수 monoid를 만들어 보자.
{% highlight scala %}
def endoMonoid[A]: Monoid[A => A] = new Monoid[A => A] {
  override def op(f: A => A, g: A => A): A => A = f compose g
  override def zero: A => A = a => a
}
{% endhighlight %}

이제 List[String] 에 접기 함수를 사용할때 (String,+) monoid 군을 적용 시킬 수 있다.  
(String,+) 은 x,y ∈ String 이고 , String 모든 원소에 항등원 "" 이 있으며 + 이항연산의 monoid law를 가지는 군이다.  
List의 fold연산은 원소를 순회 하며, 이진연산을 수행 한다.  
따라서 List[String] 집합 (String,+) monoid를 적용시키에 fold(접기) 함수가 딱 들어 맞는다.  
{% highlight scala %}
def fold[A](xs: List[A])(m: Monoid[A]): A = 
  xs.foldRight(m.zero)((a,b) => m.op(a, b)) 
{% endhighlight %}

일반적으로 fold 함수는 A => B 처럼 형식이 다르다.  
이런경우 어떻게 하면 될까? 이렇경우 A => B 인 사상를 사용 하면 된다.
{% highlight scala %}
def foldMap[A,B](xs: List[A])(f: A => B)(m: Monoid[B]): B = 
  xs.foldRight(m.zero)((a,b) => m.op(f(a),b))
{% endhighlight %}

한가지 더 예를 들어 보면 String 목록의 모든 글자의 갯수를 구하려면 어떻게 할까?
{% highlight scala %}
val xs = List("abcd","efgh","ij")
val rs = foldMap(xs)(x => x.length)(intAddMonoid)
{% endhighlight %}

위에 예에서 String => Int 함수를 좀 생각 해보아야 할 특징이 있다.  
위의 예에서의 함수는 준동형사상 이다.

# 준동형사상 & 동형사상 & 자기동형 사상
## 준동형사상
정의부터 말하자면 다음과 같다.  
군 (G,☆) 에서 군 (H,♤)로의 함수 f 가 모든 a,b ∈ G 에 대하여  
f(a) ♤ f(b) == f( a ☆ b ) 일때 함수 f를 준동형사상이라 한다.  

이를 scala code로 다시 들여다 보면 다음과 같다.
(G,☆) 에서 G : String  
(H,♤) 에서 H : IntAddMonoid[Int] 에서 Int  
☆ 이항연산은  : String의 +  
♤ 이항연산은  : IntAddMonoid의 op(Int,Int)  
그리고 이야기 주제의 주인공인 f( 준동형사상)은 String의 length 함수이다.  

{% highlight scala %}
val xs = "abcd"
val ys = "edf"
intAddMoniod.op(xs.length , ys.length) == (xs + ys).length
{% endhighlight %}

# 접을수 있는 자료구조
위에 Monoid와 fold (접기) 함수에 잘 들어 맞는다. List, Option, Either, Tree 등 접기 자료구조를 보면 타입 매개변수가 달라지고 나머지는 거의 동일하다.  
이들을 연습 삼아 공통점을 뽑아내어 trait로 작성해보면 도움이 된다.  
이는 타입매개변수가 가변적이니 이를 형식생성자를 받도록 하면 된다.  
{% highlight scala %}
trait Foldable[F[_]] {
  def foldRight[A,B](xs: F[A])(z: B)(f: (A,B) => B): B =
	foldMap(xs)(f.curried)(endoMonoid[B])(z)

  def foldLeft[A,B](xs: F[A])(z: B)(f: (B,A) => B): B = 
	foldMap(xs)(a => (b:B) => f(b,a))(dualMonoid(endoMonoid))(z)

  def foldMap[A,B](xs: F[A])(f: A => B)(m: Monoid[B]): B = 
	foldRight(xs)(m.zero)((a,b) => m.op(f(a),b))

  def concat[A](xs: F[A])(m: Monoid[A]): A = 
	foldLeft(xs)(m.zero)(m.op) 

  def toList[A](s: F[A]): List[A] = 
	foldRight(s)(List[A]())((a,b) => a::b)
}
{% endhighlight %}

# Monoid 의 관찰
위의 예제들을 보면서 느낀점은 결국 어떠한 집합에 monoid를 적용하면 집합의 원소에 결부시킨 monoid 군의 연산을 적용하여 monoid 군으로 변환 되는 것을 볼 수 있었다.  
또한 이 monoid 이항연산은 결합법칙이 성립하므로 이 연산을 병렬로 실행하더라도 문제가 없음을 안심하고 사용 할 수 있을 듯 하다.

## Monoid 의 합성
Monoid의 위업은 함성 compose에서 나타난다.  

{% highlight scala %}
val o01 = Some(Some(2))
val o02 = Some(Some(3))
{% endhighlight %}
위의 Option 자료구조에 다른 Option의 값을 합하려면 어떻게 해야 할까?  
즉 결과는 Some(Some(5))  

이때 Monoid의 합성의 위력을 보여 준다.
{% highlight scala %}
def mergeOption[O](m: Monoid[O]): Monoid[Option[O]] = {
  new Monoid[Option[O]] {
	override def zero: Option[O] = None
	override def op(o1: Option[O], o2: Option[O]): Option[O] = 
	  o1 flatMap(a => o2 map(b => m.op(a, b)))
  }
}

val o01 = Some(Some(2))
val o02 = Some(Some(3))
val rx080 = mergeOption(mergeOption(intAddMonoid)).op(o01, o02)
println(rx08)
//결과: Some(Some(5))
{% endhighlight %}

다음은 Map[String,Map[String,Int]] 일 경우  
{% highlight scala %}
val m1:Map[String,Map[String,Int]] = Map("map" -> Map("a" -> 1,"b" -> 2))
val m2:Map[String,Map[String,Int]] = Map("map" -> Map("b" -> 3))
{% endhighlight %}
위의 m1과 m2를 병합 하려면 어떻게 해야 할까?
그러니깐 결과는 다음 같이

{% highlight scala %}
val m3:Map[String,Map[String,Int] = Map("map" -> Map("a" -> 1, "b" -> 5))
{% endhighlight %}

이때 위에서 만든 monoid들을 합성하면 다음과 같다.
{% highlight scala %}
def mergeMapMonoid[K,V](m: Monoid[V]): Monoid[Map[K,V]] = {
  new Monoid[Map[K,V]] {
	override def zero: Map[K,V] = Map[K,V]()
	override def op(ma: Map[K,V], mb: Map[K,V]): Map[K,V] = 
		(ma.keySet ++ mb.keySet).foldLeft(zero) { (b1,a1) => 
          b1.updated(a1,m.op(ma.getOrElse(a1,m.zero), mb.getOrElse(a1,m.zero)))
    }
  }
}

val m1:Map[String,Map[String,Int]] = Map("map" -> Map("a" -> 1, "b" -> 2))
val m2:Map[String,Map[String,Int]] = Map("map" -> Map("b" -> 3, "c" -> 2))

val mergeM: Monoid[Map[String,Map[String,Int]]] = mergeMapMonoid(mergeMapMonoid(intAddMonoid))
val rs08: Map[String,Map[String,Int]] = mergeM.op(m1,m2)
println(rs08)
//결과: Map(map -> Map(a -> 1, b -> 5, c -> 2))
{% endhighlight %}

위의 두 경우를 합하면 Option[Map[String,Map[String,Int]]] 인 경우는 ?
원하는 monoid만  결합하면 된다.
{% highlight scala %}
val mo01 = Some(Map("map" -> Map("a" -> 1, "b" -> 2)))
val mo02 = Some(Map("map" -> Map("b" -> 3, "c" -> 2)))

val mMonoid2:Monoid[Option[Map[String,Map[String,Int]]]] =   mergeOption(mergeMapMonoid(mergeMapMonoid(intAddMonoid)))
val rs0803 = mMonoid2.op(mo01, mo02)
println(rs0803)
// 결과 
//Some(Map(map -> Map(a -> 1, b -> 5, c -> 2)))
{% endhighlight %}
위의 예제들과 같이 접을 수 있는 자료구조안에 접을 수 있는 자료구조가 또 있을 경우 위와 같이  
해당하는 자료구조에 원하는 연산을 가지고 있는 monoid 군을 합성하여 활용을 극대화 할 수 있다.  

## 여러연산을 동시에 
monoid의 compose 이외에도 접기가 가능한 어떠한 자료구조에 monoid 군을 여러게 결합하여 monoid 곱을 구할 수 가 있다.  
이렇게 함으로써 여러 연산의 결과를 동시에 처리 할 수 있다. 한번 접기에 여러 결과를...  

우선 곱은 다음과 같다. 
{% highlight scala %}
def productMonoid[A,B](m1: Monoid[A], m2: Monoid[B]): Monoid[(A,B)] = ???
{% endhighlight %}

구현 해볼까?
{% highlight scala %}
def productMonoid[A,B](m1: Monoid[A], m2: Monoid[B]): Monoid[(A,B)] = 
  new Monoid[(A,B)] {
    override zero: (A,B) = (m1.zero,z2.zero)
    override op(a: (A,B), b: (A,B)))  = (m1.op(a._1,b._1),m2.op(a._2,b._2))
  }
{% endhighlight %}

이제 목록의 길이와 제곱의 합을 동시에 구현하는 예는 다음과 같다.  
목록의 길이와 원소의 합을 동시에 구할 수 있다.
{% highlight scala %}
val xs09 = List(1,2,3,4,5,6,7,8,9,10)
val m09 = productMonoid(intAddMonoid,intAddMonoid)
val rs09 = foldMap(xs09)(a => (1,a))(m09)
  
println(rs09)
//결과:(10,55)
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
