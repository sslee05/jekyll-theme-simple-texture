---
layout: post
title: "Cats"
description: "cats"
categories: [scala-Cats]
tags: [scala,스칼라,Cats,cats, 켓츠]
redirect_from:
  - /2018/06/29/
---

> Cats.
>


* Kramdown table of contents
{:toc .toc}

현업에서 scala를 사용하고 싶지만 그러한 여건이 되지 않아 혼자 이것 저것 해보는 와중에  
la scala night 세미나 에서 엄익훈님께서 Cats 라는 것의 소개로 흥미를 가지게 되었다.  
다음의 Cats post는 Scala with Cats book를 보고 공부한 것을 정리 하는 post내용 이다.  

# Cats 설치
설치는 간단하다. 현제 시점 1.1.0 version 에 scala 2.12.6 version 으로 진행 한다.  
{% highlight scala %}
libraryDependencies += "org.typelevel" %% "cats-core" % "1.0.0"
{% endhighlight %}

혹은 g8를 이용한 template project를 사용하면 된다.  
{% highlight scala %}
sbt -Dsbt.version=1.0.4 new underscoreio/cats-seed.g8
{% endhighlight %}

# Cats 3개의 Component
Cats에는 3개의 Component 를 이루고 있다.  
1. type class 
2. type class instances
3. type class interface

- type class
어떠한 기능 행위에 대한 표현(modeling)으로 적어도 1개의 type parameter를 가지는 trait으로 표현한다.

- type class instances
특정 유형에 대한 type class의 구현체이며 Cats는 scala standard library 및 Cats 의 자료 type에 맞는 type class에 대한 구현체를 제공하고 있다.  

- type class interfaces
type class를 이용한 모든 기능행위로 주로 object interfaces 방식이나 object syntax 방식으로 제공 한다.  


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
