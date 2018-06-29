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
다음의 Cats post는 Scala with Cats book(Noel Welsh and Dave Gurnell)를 보고 공부한 것을 정리 하는 post내용 이다.  
예제의 출처는 대부분의 예제는 Scala with Cats book(Noel Welsh and Dave Gurnell) 
출처 이다.  

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
3. type class interfaces

- type class  
어떠한 기능 행위에 대한 표현(modeling)으로 적어도 1개의 type parameter를 가지는 trait으로 표현한다.

- type class instances  
특정 유형에 대한 type class의 구현체이며 Cats는 scala standard library 및 Cats 의 자료 type에 맞는 type class에 대한 구현체를 제공하고 있다.  

- type class interfaces  
type class를 이용한 모든 기능행위로 주로 object interfaces 방식이나 object syntax 방식으로 제공 한다.  

# 예제를 통해 3개의 component 보기
다음과 같은 Json lib 가 있다고 가정하고 이를 다음과 같이 선언 했다고 하자.
{% highlight scala %}
sealed trait Json 
final case class JsObject(get: Map[String, Json]) extends Json
final case class JsString(get: String) extends Json
final case class JsNumber(get: Double) extends Json
final case object JsNull extends Json
{% endhighlight %}

## type class 만들기
이제 scala type을 Json 이라는 것으로 변환하는 기능(행위)에 대하여 modeling한다면 즉 type class를 선언 하면 다음과 같다.
{% highlight scala %}
trait JsonWriter[A] {
  def format(a: A): Json
}
{% endhighlight %}

## type class instances 만들기
이를 이제 특정 type들에 대한 type class intance를 만들면 다음과 같다.  
{% highlight scala %}
object JsonWriterInstances {
  implicit val stringWriter = new JsonWriter[String] {
    def format(value: String): Json = JsString(value)
  }
  
  implicit val doubleWriter = new JsonWriter[Double] {
    def format(value: Double): Json = JsNumber(value)
  }
}
{% endhighlight %}

위는 특정 type에 대한 Json으로의 변환 기능을 나타내는 type class, 그리고 String type, Double에 대한 type class instance를 정의 했다.  Person 유형을 만들고 Person 유형에 대한 이 기능을 구현하자.  
{% highlight scala %}
case class Person(name: Sring, email: String, age: Int)

//JsonWriterIsntances 에 추가 하면 되겠다.
object JsonWriterInstances {
 //중략 ....
 
 implicit val personWriter = new JsonWriter[Person] {
   def format(person: Person): Json = person match {
     case Person(name,email,age) => 
       JsObject(Map(
         "name" -> JsString(name), 
       	"email" -> JsString(email),
        "age" -> JsNumber(age)))
   }
 }
}
{% endhighlight %}

- type class instances를 package하는 4가지 방법.  
1.object 를 선언해서 그곳에 모두 넣는 방법 (위의 예제 처럼)  
2.type class triat에 선언 하는 방법  
3.type class companion object에 선언 하는 방법  
4.type class의 type parameter type의 companion object에 선언하는 방법  

1번은 import를 해야 하고, 2번은 상속을 하면 제공되며, 3,4번은 항상 제공된다.  

- implicit 선언시 val or def  
주로 val 로 선언하는 것이 일반적이지만, def 로 해야 할때가 있다. 예를 들면 Option[A]에 대한 JsonWriter는 어떻게 선언해야 할까?  
Option[String], Option[Int],.... 에 따른 모든 유형을 제공할 수 있어야 한다. 이러할 경우 val 이 아닌 def로 선언하여 A 유형에 대한 implicit를 받아 선언하면 된다.  
{% highlight scala %}
implicit def optionWriter[A](implicit jw: JsonWriter[A]) = 
  new JsonWriter[Option[A]] {
	def write(option: Option[A]): Json = option match {
      case Some(a) => jw.write(a)
      case None => JsNull
    }
}
{% endhighlight %}

## type class interfaces 만들기
이제 type class interface를 만들어 보자.  
interface 에는 JsonWriter를 이용한 다양한 기능들을 만들면 된다. 그리고 이를 client code에서 이용하면 된다.  
이때 어떠한 방식으로 interfaces 를 client에게 제공하느냐 인데 주로 implicit를 이용한 다음과 같은 2가지 일반적인 방식으로 제공한다.  
1. interface objects
2. interface syntax

### interface object 방식   
{% highlight scala %}
object Json {
  def toJson[A](a: A)(implicit jw: JsonWriter[A]): Json = jw.format(a)
}

{% endhighlight %}

이를 이용하는 client code는 다음과 같겠다. 우선 type별 JsonWriter를 구현한 type class intances 의 implicit를 import 해야한다. 그래야 같은 scope에서  interface  method를 이용할 수 있으니 말이다.  
{% highlight scala %}
import com.sslee.cats.intro.JsonWriterInstances._

val person = Person("sslee","sslee@gmail.com",10)
val jsonValue = Json toJson person

{% endhighlight %}

### interface syntax 방식
{% highlight scala %}
object JsonWriter {
 implicit class JsonWriterOpt[A](a: A) {
   def toJson(implicit jw: JsonWriter[A]): Json = jw format a
 }
}
{% endhighlight %}
client code는 다음과 같다.  
{% highlight scala %}
import com.sslee.cats.intro.JsonWriterInstances._
import com.sslee.cats.intro.JsonWriter._

val person = Person("sslee","sslee@gmail.com",10)
val jsonValue = person.toJson
{% endhighlight %}

# 정리
Cats 에서의  3개의 주요 구성 type class, type class instances, type class interfaces 의 정의와 작성의 예를 통해 작성 방법등을 알아 보았고, type class instance의 packaging 하는 유형과 type class interfaces의 제공 방법 유형을 살펴 보았다.  


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
