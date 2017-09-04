---
layout: post
title: "scala lib 설계해 보기"
description: "scala design lib"
categories: [language]
tags: [scala]
redirect_from:
  - /2017/09/04/
---

> scala design lib.
>
> 이 내용은 저자 Paul Chiusano가 쓴 "Functional Programming in scala" 책을 공부하며 정리한 것 임.

* Kramdown table of contents
{:toc .toc}

# scala library 설계방향

1. 무엇을 하고자 하는지에 대한 생각을 가지고 간단한 예를 작성하여 그것으로 시작한다.

2. 간단한 sample 코드를 보고 하고자 하는 방향에 따른 문제점들과 그에따른 필요한 자료형식과 함수들을 고찰 해낸다.

3. 공통 관심사를 도출하고 거기에 복잡성을 더해간다.

4. 이때 자료형식과 그 원소에 작용되는 함수는 공리(정리)(수학에서 말하는 true로 간주되어지는 명제)로 부터 유도 되어야 하며 그러기 위해서는 참조투명해야 한다.


# 간단한 예제작성 해보기

lib 를 사용하면 되지만 여기서는 lib 설계 방향을 배우는 것 임으로 만들어 가는 과정을 보자.
여기서는 동시성 프로그램을 하는 lib를 만드는 것을 목표로 했을 경우의 예이다.
{% highlight scala %}
val xs = List(1,2,3,4,5)
val rs = xs.foldLeft(0)((x,y) => x + y)
println(rs)
{% endhighlight %}

위의 예는 List의 원소의 sum을 구하는 것을 foldLeft를 이용하였다.
이를 동시에 나누어서 하려면 어떻게? 간다한 예부터 만들면

{% highlight scala %}
def sum(xs: IndexedSeq[Int]): Int = {
    if(xs.size  <= 1) xs.headOption getOrElse 0
    else {
        val (l,r) = xs.splitAt(xs.length / 2)
        sum(l) + sum(r)
    }
}
{% endhighlight %}

위의 예제는 이제 노리적으로 left와 right를 분리하여 처리 할 수 있게 되었다.
(물론 실제적으로는 left오 right가 동시에 처리 되진 않지만)

# 자료형식으로 부터 도출 되어야 하는 필요한 함수 도출
sum(l)와 sum(r)를 보고 병렬처리를 나타내는 자료형식이 필요함을 도출 또 그에 따른 그 자료형식을 만들어 반환 하는 함수와, 병렬처리 결과를 얻는 함수가 필요함을 알 수 있다.
{% highlight scala %}
trait Par[+A]

object Par {
  //병렬처리할 대상을 지연으로 받아 평가할 수 있는 계산을 돌려 준다.
  def unit[A](a: => A): Par[A] = ???
  //병렬 계산에서의 결과 값을 추출 한다.
  def get[A](par: Par[A]): A = ???
}
{% endhighlight %}

위의 code을 자기고 다시 sample code를 보자
{% highlight scala %}
def sum(xs: IndexedSeq[Int]): Int = {
    if(xs.size  <= 1) xs.headOption getOrElse 0
    else {
        val (l,r) = xs.splitAt(xs.length / 2)
        val rl = Par.unit(sum(l))
        val rr = Par.unit(sum(r))

        Par.get(lr) + Par.get(rr)
    }
}
{% endhighlight %}

# 문제점 찾기
위의 sample code를 보면 Par.unit(sum(l)) 과 Par.unit(sum(r)) 로 부터 논리적으로
(각각 병렬처리가 된다고 생각하고) 동시에 처리리는 되겠지만 get 함수에 의하여 결국 left이 결과를 얻기 위해 기다리면서 right는 대기하게 된다. 이름 참조치환으로 해보면 명백히 알 수 있다.
{% highlight scala %}
Par.get(Par.unit(sum(l))) + Par.get(Par.unit(sum(r)))
{% endhighlight %}

# 해결점 찾기
get 를 호출 하지 않게 하려면 어떻게 ?
map를 만들어 보자.
{% highlight scala %}
trait Par[+A]

object Par {
  //병렬처리할 대상을 지연으로 받아 평가할 수 있는 계산을 돌려 준다.
  def unit[A](a: => A): Par[A] = ???
  //병렬 계산에서의 결과 값을 추출 한다.
  def get[A](par: Par[A]): A = ???
  //처리결과를 조합한다.
  def map2[A,B,C](a: Par[A], b: Par[B])(f: (A,B) => C): Par[C] = ???
}
{% endhighlight %}

적용해보자 이때 sum의 return type은 Par[Int]이 될 것이다.
{% highlight scala %}
def sum(xs: IndexedSeq[Int]): Par[Int] = {
    if(xs.size  <= 1) xs.headOption getOrElse 0
    else {
        val (l,r) = xs.splitAt(xs.length / 2)
        val rl = Par.unit(sum(l))
        val rr = Par.unit(sum(r))

        Par.map2(rl,rr)(_ + _)
    }
}
{% endhighlight %}

# 다시 문제점 찾기
생가해 보면 map2(left,right) 에서 목록의 left가 다 소진 되어야 right가 처리됨을 알 수 있다.
왜냐면 scala 의 function parameter는 엄격(strictness)하기 때문에 목록의 left의 depth까지 소진 map2 처리 후 최초의 left,right가 처리 될테니까.

그럼 map2의 parameter를 laziness 하게 바꾸어야 한다. 하지만 그렇게 된다면 모든 map2의 호출 client는 목록이 작은 경우에도 선택없이 항상 병렬처리 리소스를 하용해야 한다.

# 명시적 분기
병렬처리의 결과들의 조합되어야 하는 것과 병렬처리 여부를 분리한다. 따라서 map2의 인자는 strictness하게 두고 명시적으로 lazy하게 인자를 받아들이는 함수를 만들자.
{% highlight scala %}
trait Par[+A]

object Par {
  //병렬처리할 대상을 지연으로 받아 평가할 수 있는 계산을 돌려 준다.
  def unit[A](a: => A): Par[A] = ???
  //병렬 계산에서의 결과 값을 추출 한다.
  def get[A](par: Par[A]): A = ???
  //처리결과를 조합한다.
  def map2[A,B,C](a: Par[A], b: Par[B])(f: (A,B) => C): Par[C] = ???
  //명시적으로 선언함으로써 clinet는 병렬처리를 처리하고 싶을때 사용할 수 있다.
  def fork[A](a: => Par[A]): Par[A] = ???
}
{% endhighlight %}

{% highlight scala %}
def sum(xs: IndexedSeq[Int]): Par[Int] = {
    if(xs.size  <= 1) xs.headOption getOrElse 0
    else {
        val (l,r) = xs.splitAt(xs.length / 2)
        Par.map2(fork(sum(l)),fork(sum(r)))(_ + _)
    }
}
{% endhighlight %}
이제 map2의 left의 인자 depth 처리가 끝나지 않아도 최상위 left와 right가 동시에 처리 된다.


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture