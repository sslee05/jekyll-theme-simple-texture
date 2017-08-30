---
layout: post
title: "scala laziness"
description: "scala laziness"
categories: [language]
tags: [scala]
redirect_from:
  - /2017/08/30/
---

> scala laziness.

* Kramdown table of contents
{:toc .toc}

# laziness 의 목적

laziness 의 궁극적인 목적은 표현식의 서술과 평가 방법 및 시기를 분리 함으로써 재사용과 모듈성과 효율성을 증가 시킨다.


# call-by-value & call-by-name

Scala 에서  parameter는 default가 call-by-value이다. 

이것의 의미는 parameter의 대상의 결과가 인자로 넘어 간다는 것이다. parameter type 이 function 이면 실행 결과가 Fucntion의 instance 를 받는 것 이므로 혼동하지 말자

Call-by-name은 계산의 결과가 아닌 계산의 식이 넘어 가고 평가는 실제로 계산의 실행 시점에 평가가 된다.

하지만 parameter를 call-by-value가 아닌 call-by-name으로 인자를 넘길 수 있는데  이러한 기법을 scala non-strictness 또는 laziness 라 한다.

def fn(a:Int,b:Int):Int = a + b

def clientFn:Int = fn(1+2,3+4) 의 전계은 1+2,3+4의 계산 결과인 3,7이 fn의 인자로 넘어 간다. 이것이 call-by-value
반면 계산식 자체인 1+2, 3+4 가 넘어 가서 fn 의 body 안에 a + b를 만나는 곳에서 a = 1+ 2 가 대입되어 평가 되고 b = 3+ 4가 대입되어 평가 되어서 최종 (1+2) + (3 + 4)  가 되는 것이 call-by-name이다.

() => A 로 만 해야 함.
{% highlight scala %}
sealed case class Cons[+A](head:() => A,tail:() => Stream[A]) extends Stream[A]
{% endhighlight %}

# laziness 표현,호출,평가
Scala 에서 parameter를 call-by-name으로 할 수 있게 하는 방법은 2가지 표현이 있다.
a:() => A 인 명시적 thunk 표현식 ,a: => A 인 표현이 있다.
참고로 class parameter는 => A 표현이 안된다.
{% highlight scala %}
def cons[A](h: => A, t: => Stream[A]):Stream[A]= {
    lazy val head = h
    lazy val tail = t
    
    Cons(() => head,() => tail)
  }
{% endhighlight %}
위의 예에 cons method는 일반 생성자 함수와 다르다. 이를 smart 생성자라 하는데 관례적으로 class의 소문자로 시작한다. smart 생성자가 필요한 경우는 일반 생성자에서 call-by-name를 호출 할때 마다 실행되는데 그렇게 하지 않고 최소의 강제에만 실행하고 이를 memoization를 하여 다음 실행시는 그 결과 값을 가지고 실행되게 함을 하기 위하여다.

1. () => A 형태
* Parameter 선언 : a:() => A
* 평가 방법 : a()
* 호출 방법: () => A

2. a: => 형태
* Parameter 선언 : a: => A
* 평가 방법: a
* 호출 방법: a   이때 compiler가 thunk으로 둘러 쌓준다.

# scala에서 laziness 특징
* Scala laziness 표기는 사실 Function0 의 instance이다. 즉 인자가 없는 function instance인 것 이다.

* call-by-name은 실행시 마다 평가 된다. 따라서 한번만 평가 하고 이후에 평가의 결과를 사용하기 위한 방법으로 lazy keyword를 이용할 수 있다.

* thunk이란 표현식이 평가되지 않은 형태를 뜻 한다.


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture