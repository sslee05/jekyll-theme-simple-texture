---
layout: post
title: "akka-http"
description: "Akka Http"
categories: [scala-akka-http]
tags: [scala,스칼라,akka,akka-http,http]
redirect_from:
  - /2018/06/30/
---

> Akka-http.
>


* Kramdown table of contents
{:toc .toc}

akka in action 를 공부하다 akka-http부분을 만나면서 akka-http를 따로 공부해야 겠다는 생각이 들어 akka-http에 대한 공부내용을 post하기로 했다.  

# akka-http 설치
akka-http 는  akka module에서 분리 되어 독립적인 배포 사이클을 가진다. 현제 10.1.1 version 에 akka 2.5.13, scala 2.12.6 version,  으로 진행 한다.  
akka-http 는 akka-http-core와 akka-http 로 되어 있있는데 akka-http가 akka-http-core 를 dependency하게 되어 있으므로 akka-http 만 선언해도 된다.  
이는 g8 template project 가 있어 쉽게 설치 할 수 있다.  
{% highlight scala %}
sbt -Dsbt.version=1.0.4 new https://github.com/akka/akka-http-scala-seed.g8
{% endhighlight %}

# Akka-Http route
Akka-http route API 는 DSL를 제공하며, 이는 1개 이상의 Directive 로 구성되어 요청에 대한 route 처리를 기술하게 한다.  

아래에 예제는 akka-io 에 나와 있는 예제 이다.
{% highlight scala %}
object WebServer {

  def main(args: Array[String]): Unit = {

    implicit val system = ActorSystem("myFirstHttpServer")
    implicit val mat = ActorMaterializer()

    implicit val ec = system.dispatcher

    val route = path("hellow") {
      get {
        complete(HttpEntity(ContentTypes.`text/html(UTF-8)`, "<h1>hellow world akka-http</h>"))
      }
    }

    val bindFuture: Future[Http.ServerBinding] = Http().bindAndHandle(route, "localhost", 8080)

    StdIn.readLine()
    bindFuture.flatMap(serverBinding => serverBinding.unbind()).onComplete(_ => system.terminate())

  }
}
{% endhighlight %}
그리고 다음과 같은 명령으로 server를 띄운다.  
{% highlight scala %}
Macintosh:akka-http sslee$ sbt "runMain com.sslee.http.intro.WebServer"
{% endhighlight %}

그리고 command에서 다음과 같은 명령을 하면 결과는 아래와 같이 나온다. (난 HTTPie 를 설치한 상황이므로)
{% highlight scala %}
Macintosh:akka-http sslee$ http GET localhost:8080/hellow
HTTP/1.1 200 OK
Content-Length: 30
Content-Type: text/html; charset=UTF-8
Date: Sat, 30 Jun 2018 04:56:29 GMT
Server: akka-http/10.1.1

<h1>hellow world akka-http</h>
{% endhighlight %}

# akka-http spray-json
akka-http 는 client, server 간의  form data => scala type(Unmarshalling), 혹은 scala type => json string(Marshalling)으로 spray-json module를 사용한다.  
아래 예는 Item(scala type) => Json String 으로의 marshalling과 , form json data => Order(scala type)으로의 unmarshalling을 보여 준다.  
{% highlight scala %}
object SimpleMarshallWebServer {

  case class Item(name: String, id: Long)
  case class Order(items: List[Item])

  var mockDatabase = List.empty[Item]

  //route 실행 필수
  implicit val system = ActorSystem("SimpleMarshallWebServer")
  implicit val mat = ActorMaterializer()

  //Future에 필요
  implicit val ec = system.dispatcher

  //fake select database
  def getItem(id: Long): Future[Option[Item]] = Future {
    mockDatabase.find(item => item.id == id)
  }

  //fake insert database
  def saveOrder(order: Order): Future[Done] = {
    order match {
      case Order(items) =>
        mockDatabase = items ::: mockDatabase
      case _ => mockDatabase
    }

    Future { Done }
  }

  def main(args: Array[String]): Unit = {
    
    //spray-json 을 이용한 marshall, unmarshall시 필요한 암시자 
    implicit val itemFormatter = jsonFormat2(Item)
    implicit val orderFormatter = jsonFormat1(Order)

    val route: Route = get {
      pathPrefix("item" / LongNumber) { id =>
        val item = getItem(id)
        onSuccess(item) {
          //marshalling
          case Some(item) => complete(item)
          case None => complete(StatusCodes.NotFound)
        }
      }
    } ~ post {
      path("createOrder") {
        //unmarshalling
        entity(as[Order]) { order =>
          val saved: Future[Done] = saveOrder(order)
          onComplete(saved) { done =>
            complete("order created")
          }
        }
      }
    }

    val bindFuture: Future[Http.ServerBinding] = Http().bindAndHandle(route, "localhost", 8080)
    StdIn.readLine()

    bindFuture.flatMap(serverBinding => serverBinding.unbind()).onComplete(_ => system.terminate())

  }

}
{% endhighlight %}
서버 띠우기  
{% highlight scala %}
sbt "runMain com.sslee.http.intro.SimpleMarshallWebServer"
{% endhighlight %}
그리고 Order(items: List[Item]) 으로 Unmarshalling될 요청 값을 json.txt  file로 만들어 HTTPie를 이용하여 실행을 해보자.
{% highlight scala %}
//json.txt file
{"items":[{"name":"akka-book","id":45}]}

//요청실행 Order 등록 
Macintosh:akka-http sslee$ http POST localhost:8080/createOrder < ./json.txt
HTTP/1.1 200 OK
Content-Length: 13
Content-Type: text/plain; charset=UTF-8
Date: Sat, 30 Jun 2018 06:37:38 GMT
Server: akka-http/10.1.1

order created

{% endhighlight %}

등록값 조회
{% highlight scala %}
Macintosh:akka-http sslee$ http GET localhost:8080/item/45
HTTP/1.1 200 OK
Content-Length: 28
Content-Type: application/json
Date: Sat, 30 Jun 2018 06:38:13 GMT
Server: akka-http/10.1.1
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
