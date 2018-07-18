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
서버 기동  
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
{
    "id": 45,
    "name": "akka-book"
}
{% endhighlight %}

# akka http 와 akka-stream
akka-http에서 akka-stream을 쉽게 이용할 수 있으며, 이를 통해 첨부파일이나 memory size를 넘는 대량의 데이터를 처리하더라도 OOM이 발생하지 않는다. 이는 reactive stream 구조의 akka-stream 덕분이다.  아래의 예제를 통해 client browser가 Sink, server에서 무한 Random 생성기가 Source로 이는 무한이지만, 절대 OOM 이 발생되지 않음을 볼 수 있으며, 이는 client의 즉 Sink쪽에서 Source쪽으로의 backpressure 를 통해 (water mark) async 로 수위를 조절한다. 이는 akka-stream에서 이야기 한 부분이다.  
소스는 akkak-io site 에 있다.
{% highlight scala %}
object ExampleStreamWebServer {

  def main(args: Array[String]) {
    //route 실행에 필수, Stream
    implicit val system = ActorSystem("ExampleStreamWebSystem")
    implicit val mat = ActorMaterializer()

    //Future 에 필요
    implicit val ec = system.dispatcher

    val randomSource = Source.fromIterator(() => Iterator.continually(Random.nextInt))

    val route = path("random") {
      get {
        complete {
          HttpEntity(
            ContentTypes.`text/plain(UTF-8)`, randomSource.map(n => ByteString(s"$n\n")))
        }
      }
    }

    val bindFuture: Future[Http.ServerBinding] = Http().bindAndHandle(route, "localhost", 8080)
    println(s"Server online at http://localhost:8080/\nPress RETURN to stop...")

    StdIn.readLine()
    bindFuture.flatMap(serverBinding => serverBinding.unbind()).onComplete(_ => system.terminate())
  }

}
{% endhighlight %}

실행시 curl로 요청 rate 를 이용하여 요청수위를 늦춘다.  
{% highlight scala %}
Macintosh:akka-http sslee$ curl --limit-rate 50b 127.0.0.1:8080/random
-1090963985
-472554423
193702927
1496244304
....중략
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
