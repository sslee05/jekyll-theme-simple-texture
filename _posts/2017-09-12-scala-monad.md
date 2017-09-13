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
>


* Kramdown table of contents
{:toc .toc}

# 수학적 기본 지식
## function 의 조건 
- 모든 정의역의 원소에 해당하는 공역에 원소가 있다.
- 정의역의 원소 x는 항상 공역에 원소가 1개이어야 한다.(1개 초과 안됨)
위의 조건들을 만족하는 것을 구조를 보존한다 라고 한다.

# Functor(함자)

![친절한 스크린샷]({{ baseurl }}/assets/images/scala/01.png)

이미지 출처:"Professor Frisby's Mostly Adequate Guide to Functional Programming"

위의 그림을 보면 한 마디로 map를 가진 형식생성자라 할 수 있다.  
map 함수의 sygnature 는 다음과 같다.
{% highlight scala %}
def map[A,B](a: F[A])(f: A => B): F[B] 
{% endhighlight %}
위의 code는 위의 그림과 같이 함수 f: a -> b 를 받고 F[A] 를 F[B]로 변환하는 함수 즉 Functor다.  

이때 function의 조건을 만족해야 한다.  
- 모든 정의역의 원소에 해당하는 공역에 원소가 있다.
- 정의역의 원소 x는 항상 공역에 원소가 1개이어야 한다.(1개 초과 안됨)

위의 코드에서 F[A]의 구조를 보존해야 한다는 것인데 만약 F[A]가 List[Int] 이고 F[B]가 List[String] 이라면 F[B] 의 길이는 F[A]의 길이과 같을 것이며, 해당요소들은 같은 순서로 될 것이다.  

나라는 형체가 거울에 비추어 질때 projection 의 결과는 거울의 내 모습이며 이 projection를 functor라 한다. 이는 나의 형태에 모습(구조)을 그대로 반영 한다.  

이런 법칙은 다음 예와 같이 추론을 할 수 있기 때문이다. 예를 들어 2개의 Monoid를 곱으로 생성된 Monoid 또한 결합법칙을 만족하다고 추론 할 수 있다.  


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture