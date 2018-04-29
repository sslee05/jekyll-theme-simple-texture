---
layout: post
title: "scala Monoid"
description: "scala Monoid"
categories: [scala-function]
tags: [scala,스칼라,Monoid, 모노이드]
redirect_from:
  - /2017/09/08/
---

> scala Monoid.
>


* Kramdown table of contents
{:toc .toc}

# 들어가기 앞서
처음 scala 를 공부할때 새로운 언어를 배운는 것 보다 함수형으로 생각하는 것 자체가 낮설었다.  
그때 당시 Monad와 Functor, Application Functor, Traversable Functor 등 이들과 시름했던 기역들이 생각난다.  
누구나 그러하듯 바쁜 삶의 현장속에 특히나 나 같이 공공 SI를 하는 사람(애 아빠)에게는 더더욱 blog를 남긴다는 것은 웬만해선 생각 조차 들지 않는다.  
주저리는 이제 여기서 그만하고 내가 배우고 알게 된 것들을 blog에 남기기로 결심을 했다는 것이다.  
남들도 하는데 나도 한번 남겨나 보자..  

먼저  Monoid을 시작으로 앞으로 Functor 와 Monad, Applicative Functor, Traversable Functor로 작성 하겠다.

# 어렵풋한 아주 작은 수학적 이야기
우리가 고등학교때 정석 책을 보면 항상 제일 먼저나오는 집합 과 명제 부분에서 집합부분을 보면 (아주 간단한)  

어떤 원소 a 가 집합 A의 원소 일때 다음과 같이 표기 했던 것? 같다.  
a ∈  A  이렇게 ..  
그래서 자연수 집합의 어떤 원소 a 라 할때 이를 a ∈  N , 또는 정수의 집합의 어떤 원소 a라 할때  a ∈  Z 이렇게  
불르곤 했던거 같다.  

그럼 이제 "정수의 집합의 어떤 원소"를 프로그램언어 (여기서는 scala)로 어떻게 표현하지....  
{% highlight scala %}
a  :  Int  
//a ∈  Z 이 것과 위치 지정이 들어 맞네... 참고로 java는 집합 원소 라고 생각해도...

val a: String // String type(집합)의 a(원소)
val b: Option // Option type(집합)의 b(원소)
val c: MyType // MyType type(집합)의 c(원소)
.......
{% endhighlight %}

자.. 이제 어떤 집합에 대한 원소들과 이 원소들의 어떤 연산을 생각 해보자. 음...  
정수 집합 Z의 원소 2개를 더하는 연산 +  에 대하여 생각 해보면...  
1 + 2 = 3, 2 + 3 = 5 , 3 + 4 = 7 등 + 연산의 결과도 정수 이다.  이를 기호를 빌려 일반화로 나타내면 다음과 같이 나타낼 수 있다 .  

집합 F에 대하여  ✶ 의 이항 연산이 있을때 F의 원소에  ✶ 이항연산의 결과 또한 F의 원소라면 (F,  ✶)라 하고 이를 군이라 한다.  

근데 이런 군중에 Monoid 군이라는 것이 있다는데 그건 다음과 같은 조건을 만족 한다면 이를 Monoid 군이라고 한다.  

<span style="color:blue">1.*✶ 연산에 대한 항등원이 존재: x ✶ e = x , e ✶ x = x*</span>  
<span style="color:blue">2.*✶ 연산에 대한 결합법칙이 성립: (x ✶ y) ✶ z = x ✶ (y ✶ z)*</span>

# 이제 scala Monoid로 들어가 보자
{% highlight scala %}
trait Monoid[A] {
  def zero: A
  def op(a: A, b: A): A
}
{% endhighlight %}
위의 code 는 Monoid type class를 나타내며, operation에 인자들은 A 유형(A집합의 원소)이다.  
a,b ∈ A 이고 zero(항등원) ∈ A 이며 (구현은 안되어 있지만, 결합법칙이 성립하는 연산) op의 이항연산이 있다.  
이때 위의 군의 정의에 의해 Monoid[A]의 인스턴스를 monoid 라 한다.  

그럼 예를 만들어 보자.  
Int 의 (집합) + 이항연산에 대한 Monoid를 만들어 보면  
{% highlight scala %}
val intAddMonoid: Monoid[Int] = new Monoid[Int] {
  // 0 + 1 = 1 , 2 + 0 = 2
   def zero:Int = 0 

  // 1 + ( 2 + 3) = (1 + 2) + 3
  def op(a: Int, b: Int) = a + b 
}
{% endhighlight %}
String 의 + 연산에 대하여  
{% highlight scala %}
val stringappendMonoid: Monoid[String]  = new Monoid[String] {
   //"" + "a" = "a" , "b" + "" = b
   def zero: String = ""

   // ("a" + "b") + "c" = "a" +( "b" + "c")
   def op(a: String, b: String): String = a + b
}
{% endhighlight %}

Option 값의 조합연산에 대한 Monoid  
{% highlight scala %}
def optionMonoid[A]: Monoid[Option[A]] = new Monoid[Option[A]] {
  // op(None, Some("a")) = Some(a) , op(Some("a"), None) = Some("a")
  def zero:Option[A] = None

  //op(op(Some("a"), None), Some("b")) == op(Some("a"),op(Some("None"), Some("b")))
  def op(a: Option[A], b: Option[A]): Option[A] = a orElse b
}
{% endhighlight %}

마지막으로 하나만 더보자 f: A => A 집합에 대한  compose 연산 Monoid  
{% highlight scala %}
def fnMonoid [A] : Monoid[A => A] = new Monoid[A => A] {

  //op(zero, a => a) = a => a, op(a => a, zero) = a => a
  def zero: A => A = a => a

  //op(op(f,g),h) = op(f, op(g, h))
  def op(f: A => A, g: A => A) : A => A = f compose g
}
{% endhighlight %}

# Monoid와 fold 연산이 만날때
List의 fold연산은 다음과 같은 Signature를 가진다.  
{% highlight scala %}
// List의  fold는 내부적으로 foldLeft를 이용한다.
def fold[A1 >: A](z: A1)(op: (A1, A1) ⇒ A1): A1
{% endhighlight %}

이는 내부적으로 foldLeft를 이용하는데 왼쪽의 원소부터 초기값 z와 이진연산 op를 적용하며 오른쪽으로 연산을 진행한다.  
따라서 List ("a", "b","c", "d") 인경우 op(op(op(z,a),b),c) 인 형태가 된다.  
A라는 유형의 집합에 원소들의 이진 연산 op....  
웬지 Monoid 와 잘 맞을 것 같은 느낌이 든다.  

  

List[String] 에 대한 + 연산 즉 (String, +) Monoid 군을 이용하여 List 을 pattern match 를 이용하지 않고 List안에 있는 값에 +  연산을 적용하면 다음과 같이 할 수 있다.  
{% highlight scala %}
val rs01 = List("a","b","c")
rs01.fold(stringMonoid.zero)(stringMonoid.op)

//결과
abc
{% endhighlight %}

foldLeft, foldRight를 보자  
foldLeft는 아래와 같다. 왼쪽부터 오른쪽으로 접는다.  
{% highlight scala %}
//왼쪽에서 오르쪽으로 접는다.
def foldLeft[B](z: B)(op: (B, A) ⇒ B): B
//위는 A => A 아니고 결과가 B 유형이 되니 Monoid설명을 위해 잠시 다음과 같이 보자.
//A => B의 형태는 추후 설명 한다.
def foldLeft(z: A)(op: (A, A) ⇒ B): A

//오른쪽에서 왼쪽으로 접는다.
def foldRight[B](z: B)(op: (A, B) ⇒ B): B
//위는 A => A 아니고 결과가 B 유형이 되니 Monoid설명을 위해 잠시 다음과 같이 보자.
//A => B의 형태는 추후 설명 한다.
def foldRight(z: A)(op: (A, A) ⇒ A): A
{% endhighlight %}

눈치채겠지만 fold 이외에 foldLeft와 foldRight를 논의 하는 이유는 결합 법칙이 성립함을 이야기 하기 위해서이다.  
아래의 코드를 실행 해보면 결과가 같다.  
{% highlight scala %}
val xs01 = List("a","b","c")

val leftRx =  xs01.foldLeft(stringMonoid.zero)(stringMonoid.op)
val rightRx = xs01.foldRight(stringMonoid.zero)(stringMonoid.op)
println(leftRx)
println(rightRx)

//결과 
abc
abc
{% endhighlight %}

foldLeft는 왼쪽부터 시작하여 오른쪽으로 연산, foldRight는 오른쪽에서 시작하여 왼쪽으로 연산.
foldLeft와 foldRight의 결과  op(op(z,a),b) == op((op(b,z),a) 가 되었다. 이는 Monoid가 결합 법칙이 성립하기 때문이다.  

참고: 여기서는 op(a,z), op(b,z) 등 의 순서는 생각하지 말자 교환법칙과는 관계 없으므로(List의 foldRight는 내부적으로 reverse한  foldLeft를 사용하며 op연산의 매개변수 순서를 바꾼다.)  

이 Monoid는 앞으로 이야기 할 Monad, Applicative Functor, Traversable Functor 등 특정 유형의 원소들에 대한 이항 연산 적용시에 사용된다.  

# 코드를 보며 생각해 보자
{% highlight scala %}
//foldLeft나 foldRight처럼 A => B 인 경우 Monoid로 구현하려면 
//다음과 같이 f: A => B 변환 함수를 주면 된다.
def listFoldMap[A,B](xs: List[A])(f: A => B)(mi: Monoid[B]): B = 
  xs.foldLeft(mi.zero)((b,a) => mi.op(f(a),b))
{% endhighlight %}

이제 이 listFoldMap 을 이용하여 foldRight를 구현 해보자.우선 foldRight는 다음과 같다.
{% highlight scala %}
def foldRight[A,B](xs: List[A])(z: B)(f: (A,B) => B): B
{% endhighlight %}
우선 listFoldMap으로 구현하려면 Monoid가 필요하다. 어떤 집합의 이항연산을 가지는 Monoid 군이어야 하는가?...  
위에서 선언한 fnMonoid[B => B]인 Monoid를 보자. 한번 해보자.
{% highlight scala %}
def foldRight[A,B](xs: List[A])(z: B)(f: (A,B) => B): B = 
  listFoldMap(xs)(f.curried)(fnMonoid)(z)
{% endhighlight %}

Monoid는 어떤 유형의 유형의 유형의.... 값, 원소의 이진연산을 할 경우에 유용하다.  
Option[Map[String,Map[String,Int]]] 의 Int에 + 연산을 적용 하고 싶은 경우 어떻게 해야 할까?   
(아직 Monad를 이야기 하지 않았다.) 아래의 코드을 보며 생각 해보자.  
{% highlight scala %}
//우선 Option안의 값이 Monoid 유형인 경우 Monoid합성을 만들 수 있다.
def composeOptionMonoid[A](mi: Monoid[A]): Monoid[Option[A]] = new Monoid[Option[A]] {
  def zero:Option[A] = Some(mi.zero)
  def op(ma: Option[A], mb: Option[A]): Option[A] = 
      //Monad는 생각하지 말고 그냥 Option의 flatMap 과 map 함수만 생각 하자.
      ma flatMap(a => mb.map( b => mi.op(a,b)))
}

//그리고 Map 안에 Value값이 Monoid 유형인 경우의 Monoid합성을 만들어 보자.
def composeMapMonoid[K,V](:mi: Monoid[V]): Monoid[Map[K,V]] = new Monoid[Map[K,V]] {
  def zero: Map[K,V] = Map.empty
  def op(ma: Map[K,V], mb: Map[K,V]): Map[K,V] = 
    (ma.keySet ++ mb.keySet).foldLeft[Map[K,V]](Map.empty)((m,key) => m.update {
      key -> mi.op(ma.getOrElse(key, mi.zero), mb.getOrElse(key, mi.zero))
    })
}

// Option[Map[String,Map[String,Int]]] 의 안에 있는 Int의 + 연산을 적용 해보자.
val op01 = Some(Map("a" -> Map("a" -> 1, "b" -> 2,"c" -> 3)))
val op02 = Some(Map("a" -> Map("a" -> 1, "b" -> 2)))

val composeMonoid: Monoid[Option[Map[String,Map[String,Int]]]] =
  composeOptionMonoid(composeMapMonoid(composeMapMonoid(intAddMonoid)))

val rs02 = composeMonoid.op(op01,op02)
println(rs02)
//result
//Some(Map(a -> Map(a -> 2, b -> 4, c -> 3)))
{% endhighlight %}

# 참고(준동형사상)
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


다음 글에 Functor와 Monad에 대해서 작성하겠다.


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
