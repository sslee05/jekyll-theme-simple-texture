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

{% highlight scala %}
sealed case class Cons[+A](head:() => A,tail:() => Stream[A]) extends Stream[A]
{% endhighlight %}

# laziness 표현,호출,평가
Scala 에서 parameter를 call-by-name으로 할 수 있게 하는 방법은 2가지 표현이 있다.
a:() => A 인 명시적 thunk 표현식 ,a: => A 인 표현이 있다.
참고로 class parameter는 => A 표현이 안된다. () => A 로 만 해야 함.
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

# infinite stream
* laziness 덕분에 무한을 표기 할 수 가 있다. 마치 귀납법으로 무한의 집합을 정의 할 수 있는 거와 비슷하다고 해야 하나..
{% highlight scala %}
def constant[A](v:A):Stream[A] = cons(v,constant(v))
{% endhighlight %}

{% highlight scala %}
def foldRight[B](z: => B)(f:(A, => B) => B):B = {
    this match {
      case Empty => z
      case Cons(h,t) => f(h(),t().foldRight(z)(f))
    }
  }
  
def exist(p:A => Boolean):Boolean = 
    foldRight(false)((a,g) => p(a) || g)
    
def map[B](f:A => B):Stream[B] = 
    foldRight(empty:Stream[B])((a,g) => cons(f(a),g))
{% endhighlight %}

위의 foldRight를 보면 재귀가 실행되지 않는다.
f function의 B parameter 가 call-by-name 이기 때문이다.
exist 를 보면 p(a)이 true이라면 foldRight의 t().foldRight(z)(f) 이 실행되지 않는다.
map 함수 또한 cons(f(a),g) 을 보면 cons의 parameter가 call-by-name이므로 thunk으로 넘어가기 때문에 f(a)가 실행되지 않는다.

아래의 scanRight에서도 f:(A,=>B) => B 이기 때문에 loop가 먼저 실행 되지 않고 표현식 그대로 반환이 된다. 평가가 되는 시점에 loop가 실행되며 그때가 client가 scanRight를 호출한 시점에 넘긴 f(:a,b) => a + b 에서 b 부분을 만날때 t().foldRight(z)(f) 가 실행 된다.

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture