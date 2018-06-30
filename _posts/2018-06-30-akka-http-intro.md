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
그리고 command에서 다음과 같은 명령을 하면 결과는 아래와 같이 나온다. (난 httpie 를 설치한 상황이므로)
{% highlight scala %}
Macintosh:akka-http sslee$ http GET localhost:8080/hellow
HTTP/1.1 200 OK
Content-Length: 30
Content-Type: text/html; charset=UTF-8
Date: Sat, 30 Jun 2018 04:56:29 GMT
Server: akka-http/10.1.1

<h1>hellow world akka-http</h>
{% endhighlight %}


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
