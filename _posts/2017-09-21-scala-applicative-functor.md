---
layout: post
title: "scala Applicative Functor"
description: "Applicative Functor"
categories: [language]
tags: [scala]
redirect_from:
  - /2017/09/08/
---

> scala Applicative Functor


* Kramdown table of contents
{:toc .toc}

# Applicative Functor
Monad에서 이야기 하면서 box에서 값을 꺼내고, 다른 box에서 사상,값을 꺼내서 적용할 수 있게 하는 구조가 Applicative Functor라 했다.  
Monad가 flatMap으로 대표 된다면, Applicative Functor는 map2라는 것으로 대표된다.  
이들은 각각 장단점이 있으며 그 차이점들을 찾아가 보자.

# Applicative Functor trait
## Applicative Functor trait 정의 
Applicative functor는 다음의 함수 unit 항등함수와 map2의 함수를 가진다.  
또한 map함수를 map2와 unit으로 구현함으로써 Applicative functor 또한 Functor의 하위 구조가 될 수 있음을 알 수 있다.  
따라서 Applicative functor도 Functor의 성질인 구조적 보존의 법칙을 지켜야 한다.  
{% highlight scala %}
trait Applicative[F[_]] extends Functor[F] {
  // 기본수단 
  def unit[A](a: => A): F[A]
  def map2[A,B,C](fa: F[A], fb: f[B])(f: (A,B) => C): F[C]
  
  //파생 function
  def map[A,B](fa: F[A])(f: A => B): F[B] = 
    map2(fa,unit())((a,_) => f(a))
}
{% endhighlight %}

## Applicative Functor trait의 또다른 표현 
Applicative Functor는 map2를 apply(적용하다) 함수를 기본수단하여 구현 할 수 있다.
{% highlight scala %}
trait Applicative[F[_]] extends Functor[F] {
  // 기본수단 
  def unit[A](a: => A): F[A]
  def apply[A,B](fab: F[A => B])(fa: F[A]): F[B]

  //파생조합기 
  def map[A,B](fa: F[A])(f: A => B): F[B] = 
	apply(unit(f))(fa)
  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C] = 
	apply(map(fa)(f.curried))(fb)
}
{% endhighlight %}

모든 Moand는 Functor 이며 Applicative Functor라 했다. 그럼 flatMap으로 map과 map2를 구현 할 수 있다.
{% highlight scala %}
trait Monad[F[_]] extends Applicative[F] {
  //monad의 기본 조합기 
  def unit[A](a: => A): F[A]
  def flatMap[A,B](fa: F[A])(f: A => F[B]):F[B]

  //퍄생조합기 
  override def map[A,B](fa: F[A])(f: A => B): F[B] = 
    flatMap(fa)(a => unit(f(a)))
  override def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C] = 
    flatMap(fa)(a => map(fb)(b => f(a,b)))
}
{% endhighlight %}

# Monad와 Applicative Functor의 차이점
monad는 구조를 이전의 funtion effect의 결과에 따라 동적으로 선택할 수 있다. 즉 이전의 계산결과가 그다음의 계산 결과에 영향을 미친다.  
반면 Applicative는 두 연산이 서로 상관없이 독립적으로 실행되며 그냥 function effect를 차례로 적용할 뿐이다.  
{% highlight scala %}
val AF = Applicative.optionAF

val users = Map("sslee" -> "이상석", "iwlee" -> "이일웅" ,"jkwhang" -> "황진규")
val lectures = Map("이상석" -> "scala", "이일웅" -> "docker", "황진규" -> "anguler")
val dates = Map("이상석" -> "Monday", "이일웅" -> "Tuesday", "황진규" -> "Wednesday")

val rs01 = users get "sslee" flatMap(a => AF.map2(lectures get a, dates get a){
 (l,d) => s"$a 님의 강의 과목은 $l 이며 요일은 $d 입니다."
})
val rs02 = users get "sslee05" flatMap(a => AF.map2(lectures get a, dates get a){ 
  (l,d) => s"$a 님의 강의 과목은 $l 이며 요일은 $d 입니다."
})
val rs03 = AF.map2(lectures get "이상석" , dates get "이상석")((l,d) => {
  s"$l 강의는 $d 요일에 있습니다"
})
{% endhighlight %}
rs01 의 flatMap에서는 users 정보에 결과가 있을 경우에만 이후의 작업인 AF.map2 가 실행된다.  
flatMap의 결과 조차 없다면 이후 작업은 가지도 않는다.  
rs03의 map2는 lectures 조회와, dates 조회가 각각 독립적으로 실행이 된다.  
즉 lectures 의 실행은 dates의 실행에 아무런 영향을 주지 않는다.  

위의 이런 Applicative Functor의 특징은 validation 에 적합한 예를 가진다.  
web service 에서 사용자가 입력한 form 정보의 field에 따른 모든 검증결과를 담을 수 있기 때문이다.  
반면 Monad인 flatMap으로 한다면 1번째 검증이 성공해야만 다음 검증을 실행할 수 있다.  

### function effect
Option,List,Map 등 형식생성자의 자료구조는 값을 포함하는 것 이외의 기능를 추가로 가진다. 이런 추가적 기능을 Function effect라 한다.  






[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
