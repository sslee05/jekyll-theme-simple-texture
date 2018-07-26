---
layout: post
title: "akka-http HTTP model"
description: "Http model"
categories: [scala-akka-http]
tags: [scala,스칼라,akka,akka-http,http model]
redirect_from:
  - /2018/07/27/
---

> HTTP Model.
>


* Kramdown table of contents
{:toc .toc}

# Http Model
akka HTTP model 은 Http Request, Response, Common Header를 case class로 modeling한 immutable 한 type 이다. 이는 akka-http-core에 있다.  
Http Model 에 관련된 package 는 
{% highlight scala %}
import akka.http.scaladsl.model._
{% endhighlight %}
이는  HttpRequest, HttpResponse, HttpMethod, ContentType, Uri, HttpCharset,  등이 있다.

# HttpRequest
HttpRequest 와 HttpResponse는 HttpMessage를 표현하는 기본 case class 이다.  
사실 code는 case class로 되어 있지는 않지만, case class로 생기는 자동 method들을 구현 했으며, companion object의 apply method는 default 인자로 채워져 있다. 인자는 다음 인자로 구성되어 진다.HttpMethod,Uri,immutable.Seq[HttpHeader],RequestEntity,protocol: HttpProtocol
{% highlight scala %}
def apply(
  method:   HttpMethod                = HttpMethods.GET,
  uri:      Uri                       = Uri./,
  headers:  immutable.Seq[HttpHeader] = Nil,
  entity:   RequestEntity             = HttpEntity.Empty,
  protocol: HttpProtocol              = HttpProtocols.`HTTP/1.1`)
{% endhighlight %}
위의 code는 HttpRequest object의 apply method이다.  
따라서 다음과 같이 할 수 있다.  
{% highlight scala %}
import akka.http.scaladsl.model._
import HttpMethods._
import akka.util.ByteString

val uri = Uri("/abc")
HttpRequest(GET, uri)
  
HttpRequest(GET, uri="index")
  
val data = ByteString("abc")
HttpRequest(GET,uri,entity=data)

import headers.BasicHttpCredentials
mport MediaTypes._
import HttpProtocols._
import HttpCharsets._
  
val authorization = headers.Authorization(BasicHttpCredentials("user","pass"))
HttpRequest(
 GET,
 uri = "/user",
 entity = HttpEntity(`text/plain` withCharset `UTF-8`, data),
 headers = List(authorization),
 protocol = `HTTP/1.0`
)
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
