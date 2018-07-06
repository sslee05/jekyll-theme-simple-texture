---
layout: post
title: "Cats 사용해 보기"
description: "cats 사용해보기"
categories: [scala-Cats]
tags: [scala,스칼라,Cats,cats, 켓츠]
redirect_from:
  - /2018/07/05/
---

> Cats.
>


* Kramdown table of contents
{:toc .toc}

저번시간에 type class 와, type class instance, type class interface에 대하여 알아 보앗다. 이번에는 Cats에서 제공하는 type class, instances, inferfaces 를 사용하는 예의 시작으로 Show와 Eq의 예를 알아 보자.  

# packaing
cats 의 type class 가 있음  
import cats._  
혹은 지정시  
import cats.Show  
  
cats의 type instances 들  
import cats.instances.all._  
혹은 지정시  
import cats.instances.int._  
  
cats의 type class interface syntax  
import cats.syntax._  
혹은 지정시 
import cats.syntax.eq._  

instances 와 syntax을 같이 지정시  
import cats.implicits._  

아래의 Show, Eq를 통한 사용 예를 보면 알 수 있다.  

# Show
Show는 저번시간의 Printable 과 같은 type class 이다. 이는 A => String 이라고 생각 하면됨. 
cats 라는 package에 Show 의 type class가 선언 되어있으며,
type class의  apply method가 있으며 이는 implicit로 Show의 해당하는 type에 해당하는  instance가 implicit가 scope에 있어야 한다.  
이런 방식은 Show만이 아닌 모든 Cats에 있는 거진 모든 유형이 이렇게 된다고 생각 하면 됨.
{% highlight scala %}
import cats._
import cats.instances.int._
  
// type 에 해당흔 implicit type class instances가 필요
val intShow = Show[Int]
intShow show 123
{% endhighlight %}

interface syntax를 이용하는 경우  
{% highlight scala %}
import cats.instances.all._
import cats.syntax.all._

123.show
{% endhighlight %}

instance 와 syntax를 한번에 import 시 
{% highlight scala %}
import cats.implicits._
123.show
{% endhighlight %}

# Custom type 지원 
Cats 는 type class의 object에 사용자 type에 해당하는 유형의 type class instances를 생서할 수 있게 해주는 helper method들이 있다.
아래은 Show object의  method 이다.
{% highlight scala %}
def show[A](f: A => String): Show[A] = new Show[A] {
  def show(a: A): String = f(a)
}
{% endhighlight %}
Date용 Show instance 를 만들어 보자
{% highlight scala %}
import cats._
implicit val dateShowInstance: Show[Date] = 
 Show.show(date => s"${date.getTime}ms")

import cats.syntax.all._
val date = new Date()
println(date.show)
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
