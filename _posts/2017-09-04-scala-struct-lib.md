---
layout: post
title: "scala laziness"
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
//병렬처리할 대상을 지연으로 받아 평가할 수 있는 계산을 돌려 준다.
def unit[A](a: => A): Par[A] = ???
//병렬 계산에서의 결과 값을 추출 한다.
def get[A](par: Par[A]): A = ???
{% endhighlight %}
[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture