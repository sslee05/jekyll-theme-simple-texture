---
layout: post
title: "scala Monad (1)"
description: "scala Monad (1)"
categories: [monad와 친구들]
tags: [scala,Monad,스칼라,모나드]
redirect_from:
  - /2017/09/08/
---

> scala Monad


* Kramdown table of contents
{:toc .toc}

# 추상화
함수적 프로그램밍에 나오는 공통의 패턴을 추상화 함으로써 코드의 중복을 줄이고 개념적 통합을 이룰 수 있다. 이는 서로 다른 문맥의 서로 다른 해법들에서 공통구조를 인식 할 수 있다.  

# 이야기의 시작
String,Int,Boolean,... 이것에 function을 적용하는 것은 아래와 같이 쉽고 직관적이다.
{% highlight scala %}
val a = 2
def double(v: Int): Int = v * v
val rx = double(2)
//결과: 4
{% endhighlight %}

만약 함수 double에 List[Int],Option[Int]등을 넣으면 어떻게 계산되어야 할까?...  

# typeclass
동작을 정의하는 일종의 인터페이스.  
유형이 유형 클래스의 일부인 경우, 이는 유형 클래스가 설명하는 동작을 지원하고 구현함을 뜻 한다.  
java의 interface라 생각.

# map 함수에 대한 고찰
어떠한 function 이 있다고 하자.  
{% highlight scala %}
def double(v: Int) : Int = v * v

// Int 유형인 3을 적용 
val rx = double(3)
//결과: 9
{% endhighlight %}

그리고 Int 유형을 가지는 List[Int]가 있다고 하자. 이를 box 라고 부르기로 하자.
{% highlight scala %}
val xs = List(1,2,3,4,5)
{% endhighlight %}

이제 이 box를 위의 double 함수에 적용해보자 
{% highlight scala %}
val rs = double(xs) // ERROR
{% endhighlight %}

Int 유형에는 잘 적용되는 double 함수가, Int 유형을 감싼 box에는 적용되지 않는다.  
이 함수를 box에 어떻게 적용해야 하나?  
map함수를 생각 해보자  
{% highlight scala %}
def mapIntList(xs: List[Int])(f: Int => Int): List[Int] = ???
{% endhighlight %}

Int 유형을 감싼 List box를 기존의 Int 유형에 적용할 수 있는 double이라는 함수를 받아 적용할 수 있다.  

예제 하나더...
{% highlight scala %}
def twice(v: String) : String = v + v

val rx = twice("a")
//결과 aa

val xs = List("a","b","c")
twice(xs) // ERROR

def mapStringList(xs: List[String])(f: String => String): List[String] = ???
val rx = mapStringList(xs)(twice)
//결과:List("aa","bb","cc")
{% endhighlight %}

map은 box에 대하여 함수를 어떻게 적용해야 하는지를 알고 있다.  
위에서 한것들을 그림으로 나타내면 다음과 같다.  
[아래 그림들 이미지 출처:] [link](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)  

![functor]({{ baseurl }}/assets/images/scala/value_apply.png)  
유형을 function에 적용  
![functor]({{ baseurl }}/assets/images/scala/value_and_context.png)  
box  
![functor]({{ baseurl }}/assets/images/scala/no_fmap_ouch.png)  
box를 function에 적용 
![functor]({{ baseurl }}/assets/images/scala/fmap_apply.png)  
box를 map에 적용  
![functor]({{ baseurl }}/assets/images/scala/fmap_just.png)  
map이 한 것  
  
예제는 List[Int], List[String], Option[String], Option[Int] 등에 적용할 map를 만들 수 있음을 보여 준다.  

# Functor
## Functor 정의
위에 예제를 보면 box에 따라, 유형에 따라 일일이 만들 어야 한다.  
이들의 공통점을 찾아 일반화 하여 typeclass로 표현해 보자.
{% highlight scala %}
trait Functor[F[_]] {
  def map[A,B](a: F[A])(f: A => B): F[B]
}
{% endhighlight %}

![functor]({{ baseurl }}/assets/images/scala/01.png)  
[이미지 출처:mostly-adequate-guide 에서]  

이것이 Functor 다.  
이를 아래와 같이 좀 수정하여 보면 위의 그림을 좀더 쉽게 이해 할 수 있다.  
{% highlight scala %}
def map[A,B](f: A => B)(ma: F[A]): F[B] = ???
val fn = map _
{% endhighlight %}

위의 code는 위의 그림과 같이 함수 f: a => b 를 받고 F[A] 를 받아 F[B]로 변환하고 있다. 
**즉 map함수를 이용하여 box에 따라 함수를 어떻게 적용하지를 알려주는 datatype이 Functor다.**  
이 map의 input/output은   (a => b) => (F[A] => F[B]) 와 같다.
즉 functor는 box(context 상태)에 담긴 값을 꺼내어 function을 적용한다.

## Functor의 조건
functor는 function의 조건을 만족해야 한다.  
- 모든 정의역의 원소에 해당하는 공역에 원소가 있다.
- 정의역의 원소 x는 항상 공역에 원소가 1개이어야 한다.(1개 초과 안됨)

위의 코드에서 F[A]의 구조를 보존해야 한다는 것인데 만약 F[A]가 List[Int] 이고 F[B]가 List[String] 이라면 F[B] 의 길이는 F[A]의 길이과 같을 것이며, 해당요소들은 같은 순서로 될 것이다.  

거울에 비친 자신의 모습을 볼때, 거울에 보이는 모습은 나를 project하여 비추어진 결과이다.  
이는 나의 형태에 모습(구조)을 그대로 반영 한다. 이는 Functor의 조건과 같다.

# map의 법칙 관찰
def id(a: A): A = a 인 항등함수가 있다.  
def unit(a: A): Option[A] = Some(a) 인 함수가 있다.  

다음과 같은 법칙을 발견 할 수 있다.
{% highlight scala %}
map(unit(x))(f) == unit(f(x))
//f를 항등함수로 치환하면
map(unit(x))(id) == unit(id(x))
map(unit(x))(id) == unit(x)
//unit(x)를 y 로 치환하면 
map(y)(f) == y
{% endhighlight %}
위의 map의 성질은 임의의(Any) 에 모두 적용이 된다.

# box 라 부르기로 한 것
위의 box는 형식생성자를 말한다. 즉 어떤 유형의 행동을 기술하는 typeclass의 datatype을 뜻한다.  
List,Option 등등..  
box가 List라면  
{% highlight scala %}
val listF = new Functor[List] {
  def map[A,B](xs: List[A])(f: A => B): List[A] = xs map f
}
{% endhighlight %}

xs map f 는 List 역시 Functor임을 알 수 있게 해준다.  
즉 F[ _ ] 또한 F[ _ ] 에 맞는 function적용 방법을 가지고 있고 이 또한 Functor였다.

한가지 더 function에  function을 적용 해보면  
{% highlight scala %}
def map[A,B,C](f: A => B)(g: B => C): A => C = g compose f
{% endhighlight %}
함수의 합성이 된다. Function 또한 Functor 임을 알 수 있다.

# Monad
## Monad 정의
Monad는 box(context 상태)에 있는 값을 꺼내어 주어진 함수를 적용해서 box에 담는 것 이다.  
Functor는 box(context 상태)에 있는 값을 꺼내어 주어진 함수를 적용하는 방법  

위에 정의로만 보면 모든 Monad는 Functor임을 알 수 있다.  
하지만 모든 Functor가 Monad는 아니다.  

이른 scala trait으로 모델링 하면 다음과 같다.
{% highlight scala %}
trait Monad[F[_]] {
  def unit[A]:(a: => A): F[A]
  def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B]
}
{% endhighlight %}
Functor와 차이점은 functor의 map f가 A => B 이였다면 monad의 f는 A => F[B] 이다.  
이는 정의에서와 같이 box안에 원소를 꺼내어 원소에 f를 적용하여 다시 box에 담는 형태다.  

## Monad law
Monad는 또한 Monoid처럼 법칙을 가지고 있어야 한다. 다음은 그 법칙 이다.  
- bind operator는 결합법칙을 만족 해야 한다.
- left 항등법칙을 만족해야 한다.
- right 항등법칙을 만족해야 한다.

따라서 다음과 같은 조건을 scala 로 표현하면 다음과 같다.
{% highlight scala %}
//결합법칙 (m bind f) bind g == m bind (a => f(a) bind g)
ma flatMap f flatMap g == ma flatMap(a => f(a) flatMap(g))

//left 항등법칙
unit(x) flatMap f == f(x)

//right 항등법칙
ma flatMap unit == ma
{% endhighlight %}

# Applicative Functor & Monad
functor를 보면 상자에서 값을 꺼내어 function을 적용했다.  
{% highlight scala %}
def map[A,B](a: F[A])(f: A => B): F[B] = unit(f(a))

val xs = Some(2)
def double(v: Int): Int = v * v

val rs = map(xs)(double)
//결과: Some(4)
{% endhighlight %}
여기까지가 Functor에서 해왔던 것 이다.  
그림으로 나타 내면 아래와 같이...  
![functor]({{ baseurl }}/assets/images/scala/fmap.png)  
이제 f 적용함수 또한 box에 있다고 하자  
그리고 이를 기존 functor처럼 적용 해보자.  
{% highlight scala %}
def map[A,B](a: F[A])(f: A => B): F[B] = unit(f(a))

val xs = Some(2)
val fb = Some((a:Int) => a * a)

val rs = map(xs)(fb) //ERROR
{% endhighlight %}
이를 적용하기 위해서는 다음과 같은 function 이 필요하다.  

{% highlight scala %}
val xs = Some(2)
val fb = Some((a:Int) => a * a)

def map2[A,B](ma: Option[A])(mb: Option[A => B]): Option[B] = 
  ma flatMap(a => mb map(b => b(a)))
  
val rs = map2(xs)(fb)
//결과: Some(4)
{% endhighlight %}

이를 좀더 일반화 해서 다시 작성하면 다음과 같다.  
{% highlight scala %}
def map2[A,B,C](ma: F[A], mb: F[B])(f: (A,B) => C): F[C] 
{% endhighlight %}
이를 구현 하면 다음과 같다.  
{% highlight scala %}
def map2[A,B,C](ma: F[A], mb: F[B])(f: (A,B) => C): F[C]  = 
  ma flatMap(a => mb map(b => f(a,b))) 
{% endhighlight %}
구현의 안을 보면 flatMap(bind 라고 함) 와 map(이건 functor 에서) 를 사용하고 있다.  
위와 같이 flatMap은 map를 이용하여 mb box에 있는 function 혹은 값을 거내어 ma의 있는 값과 계산을 할 수 있는 point를 제공할 수 있다.  
이처럼 **bind 함수와 map 함수를 적용하여 box안의 함수를 적용하는 것을 Application Functor라 한다.**  
그리고 **Monad는 bind 함수와 map 함수를 적용하여 box안의 함수를 적용하고 다시 box를 반환하는 것이 Monand 이다.**  
**이때 monod law 3가지 를 지켜야 한다.**

따라서 위에 scala code로 Monad를 정의한 부분에 최소한의 operation set은  
unit,flatMap과 map이 있어야 할 것 이다. 근데 map은 Functor의 operation set이므로  
다음과 같이 정의 할 수 있다.
{% highlight scala %}
trait Monad[F[_]] extends Functor[F]{
  def unit[A]:(a: => A): F[A]
  def flatMap[A,B](ma: F[A])(f: A => F[B]): F[B]
  def map[A,B](ma: F[A])(f: A => B): F[B]
}
{% endhighlight %}
모든 Applicative functor 는 functor이다.  
모든 Monad는 Applicative functor 다.  
하지만 역은 항상 성립하지 안는다.  

자 이제 지금까지 한것을 그림으로 나타내면 다음과 같다  
function이 box에 담김  
![functor]({{ baseurl }}/assets/images/scala/function_and_context.png)  
map2를 적용함.  
![functor]({{ baseurl }}/assets/images/scala/applicative_just.png)  

3개,4개,5개  ... n 개도 할 수 있다.  
{% highlight scala %}
def map3[A,B,C,D](ma: F[A], mb: F[B], mc: F[C])(f: (A,B,C) => D): F[D] = 
  ma flatMap(a => mb flatMap(b => mc map(c => f(a,b,c))))
{% endhighlight %}

모든 Monad는 (Applicative Functor이므로) monad flatMap monad flatMap monad ...  
를 할 수 있다.  
이를 그림으로 나타내면 다음과 같다.  
![functor]({{ baseurl }}/assets/images/scala/monad_chain.png)  
이를 Monad chain 이라 한다.  

# 정리 
![functor]({{ baseurl }}/assets/images/scala/recap.png)  

- Functor:  
**map함수를 이용하여 box에 따라 함수를 어떻게 적용하지를 알려주는 typeclass의 datatype이  Functor다**
- Applicative functor:  
**bind 함수인 flatMap과 map 함수를 적용하여 box안의 함수를 적용하는 typeclass의 datatype이 Application Functor라 한다**
- Monad:  
**Monad는 bind 함수와 map 함수를 적용하여 box안의 함수를 적용하고 다시 box를 반환하는 것이 Monand 이다.**
- **그리고 이들의 operate들은 law를 가지고 있다.**


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
