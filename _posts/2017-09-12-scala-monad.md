---
layout: post
title: "scala Monad2"
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
## function 의 조건 ( 구조를 보존 한다.)
- 모든 정의역의 원소에 해당하는 치역의 원소가 있다.
- 정의역의 원소 x는 항상 1개의 치역의 원소이다.

# Functor(함자)

![친절한 스크린샷]({{ baseurl }}/assets/images/scala/01.png)

이미지 출처:"Professor Frisby's Mostly Adequate Guide to Functional Programming"

위의 그림을 보면 한 마디로 map를 가진 형식생성자라 할 수 있다.  
map 함수의 sygnature 는 다음과 같다.
{% highlight scala %}
def map[A,B](a: F[A])(f: A => B): F[B] 
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture