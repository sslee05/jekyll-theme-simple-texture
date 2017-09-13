---
layout: post
title: "scala Monad"
description: "scala Monad"
categories: [language]
tags: [scala]
redirect_from:
  - /2017/09/08/
---

> scala Monad.  
> 나도 배우는 입장에서 공부한 것을 나름 내방식으로 이해 한것이니 틀린 것 이나 잘 못 이해한 것이 있을 수 있다  


* Kramdown table of contents
{:toc .toc}

# 이야기의 시작
String,Int,Boolean,... 이것에 function을 적용하는 것은 아래와 같이 쉽고 직관적이다.
{% highlight scala %}
val a = 2
def double(v: Int): Int = v * v
val rx = double(2)
//결과: 4
{% endhighlight %}

만약 List[Int],Option[Int]등 에 함수 double를 넣으면 어떻게 계산되어야 하나?
문제는 이것으로 부터 시작 된다.

# typeclass
동작을 정의하는 일종의 인터페이스.  
유형이 유형 클래스의 일부인 경우, 이는 유형 클래스가 설명하는 동작을 지원하고 구현 함을 뜻 한다.  
자바 인터페이스라고 생각.

# map 함수에 대한 고찰
어떠한 function 이 있다고 하자.  
{% highlight scala %}
def double(v: Int) : Int = v * v

// Int 유형인 3을 적용 
val rx = double(3)
//결과: 9
{% endhighlight %}

그리고 Int 유형을 가지는 List[Int]가 있다고 하자. 이를 boxed 라 가칭하자.
{% highlight scala %}
val xs = List(1,2,3,4,5)
{% endhighlight %}

이제 이 boxed를 위의 double 함수를 적용해보자 
{% highlight scala %}
val rs = double(xs) // ERROR
{% endhighlight %}

Int 유형에는 잘 적용되는 double 함수가 Int 유형을 감싼 box에는 적용되지 않는다.  
이 함수를 box에 어떻게 적용해야 하나?  
map함수를 생각 해보자  
{% highlight scala %}
def mapIntList(xs: List[Int])(f: Int => Int): List[Int] = ???
{% endhighlight %}

Int 유형을 감싼 List box를 기존의 Int 유형에 적용할 수 있는 double이라는 함수를 받아 적용하고 있다.  

예제 하나더
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

이것이 Functor 다.  
![친절한 스크린샷]({{ baseurl }}/assets/images/scala/01.png)  
[이미지 출처:mostly-adequate-guide 에서]  
위의 code는 위의 그림과 같이 함수 f: a -> b 를 받고 F[A] 를 F[B]로 변환하는 함수 즉 Functor다.  
__즉 box에 따라 함수를 어떻게 적용하지를 알려주는 typeclass가 Functor다.__  
이 map은 결국  (a => b) => (F[A] => F[B]) 의 기능을 하는 typeclass다.

## Functor의 조건
functor는 function의 조건을 만족해야 한다.  
- 모든 정의역의 원소에 해당하는 공역에 원소가 있다.
- 정의역의 원소 x는 항상 공역에 원소가 1개이어야 한다.(1개 초과 안됨)

위의 코드에서 F[A]의 구조를 보존해야 한다는 것인데 만약 F[A]가 List[Int] 이고 F[B]가 List[String] 이라면 F[B] 의 길이는 F[A]의 길이과 같을 것이며, 해당요소들은 같은 순서로 될 것이다.  

나라는 형체가 거울에 비추어 질때 projection 의 결과는 거울의 내 모습이며 이 projection를 functor라 한다.  
이는 나의 형태에 모습(구조)을 그대로 반영 한다.  

# box(가칭)
위의 box를 List의 예로 보면 
{% highlight scala %}
val listF = new Functor[List] {
	def map[A,B](xs: List[A])(f: A => B): List[A] = xs map f
}
{% endhighlight %}

xs map f 는 List 역시 Functor임을 알 수 있게 해준다.  
즉 F[ _ ] 또한 F[ _ ] 에 맞는 function적용 방법을 가지고 있고 이 또한 Functor였다.

한가지 더 function 이라는 box에 function을 적용 해보면  
{% highlight scala %}
def map[A,B,C](f: A => B)(g: B => C): A => C = g compose f
{% endhighlight %}
함수의 합성이 된다. Function 또한 Functor 임을 알 수 있다.






[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture