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
Http Model 에 관련된 package는 아래와 같다.
{% highlight scala %}
import akka.http.scaladsl.model._
{% endhighlight %}
이는  HttpRequest, HttpResponse, HttpMethod, ContentType, Uri, HttpCharset,  등이 있다.

# HttpRequest
HttpRequest 와 HttpResponse는 HttpMessage를 표현하는 기본 case class 이다.  
사실 code는 case class로 되어 있지는 않지만, case class로 생기는 자동 method들을 구현 했으며, companion object의 apply method는 default 인자로 채워져 있다. 인자는 다음 인자로 구성되어 진다.  
- HttpMethod  
- Uri  
- Seq of headers  
- Entity body  
- protocol  
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

# HttpResponse
HttpResponse 구성  
- StatusCode  
- Seq of Headers  
- Enity body data  
- protocol  

{% highlight scala %}
//Simple ok
HttpResponse(200)
import akka.http.scaladsl.model._
import StatusCodes._
  
//NotFound 404
HttpResponse(NotFound)
//NotFound 404 with body data
HttpResponse(NotFound, entity="Not found of Resource")
  

//response redirect by header 
val redirect = headers.Location("http://www.domain.com/login.do")
HttpResponse(Found, headers = List(redirect))
{% endhighlight %}

# HttpEntity
Message 의 data byte를 수신하거나 전송에 대한 model 를 나타낸다.  
이는 5가지의 Enity가 있다.  

## HttpEntity.Strict
data 양이 적고 이미 알려진 유형인 String,ByteString 인 경우 

## HttpEntity.Default
컨텐츠 데이터는 Source\[ByteString,_\] 로 표시
Stream Source에 의해 data가 생성되고, 데이터 크기를 알 수 있는 경우

## HttpEntity.Chunked 
컨텐츠 데이터는 Source\[HttpEntity.ChunkStreamPart\] 로 표시
길이가 알려지지 않은 경우의 Chunked 사용시 이때 chunk 분할전송에 대한 encoding 지원 가능시 Chunked 그렇지 않는 경우 CloseDelimited를 사용

## HttpEntity.CloseDelimited
컨텐츠 데이터는 Source\[ByteString,_\] 로 표시
닫힌 연결인 서버에서만 사용으로 Response 에서만 사용 가능

## HttpEntity.IndefiniteLength
Multipart.BodyPart에서 사용하기 위해 지정되지 않은 길이의 스트리밍 엔터티

## HttpEntity의 하위 type
하위 유형에 따른 처리를 다르게 할 경우 하위 유형을 알아야 하지만 대부분 하위 유형를 알 필요는 없다.  
HttpEntity.dataBytes 를 통해 Enity data에 access할 수 있는 Source\[ByteString,_\]를 얻어 처리한다.  

## Limit message entity length
설정 파일에 max-content-length 으로 global 설정하는 할 수 있는데 이는 모든 request 요청에 대하여 적용이 된다.  
만약 개별로 설정하고 싶은경우 HttpEntity에 withSizeLimit method를 이용하여 설정하면 된다.  
설정을 통하든 method로 설정하든 설정에 위배되는 경우 실채화시에 Exception이 발생한다.  




[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
