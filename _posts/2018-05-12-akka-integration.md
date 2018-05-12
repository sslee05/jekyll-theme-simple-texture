---
layout: post
title: "akka Integration"
description: "akka Integration"
categories: [scala-akka]
tags: [scala,스칼라,akka,아카,Integration,EIP,아카시스템통합]
redirect_from:
  - /2018/05/12/
---

> akka FSM.
>


* Kramdown table of contents
{:toc .toc}

# Enterprise Integration Pattern
이제는 많은 곳에서 여러 서비스 시스템간의 정보교환를 필요로 하고 있다.  
이를 위해서는 다른 시스템간의 interface가 필요한데 이는 두 시스템간에 어떤 메시지를 어떻게 보낼지에 대한 규약를 포함 한다.  
따라서 메시지 규약과 그에 따르는 메시지에 대한 변환이 필요하고 어떻게 보낼지에 대한 전송방법 (HTTP, TCP, Queue, file등)에 대한 것도 신경써야 한다.  
이런 방법에 따라 기존 application에 수정, 변경이 일어 난다면 좋지 않을 것이다. 또한 해당 application내의 로직에서는 data가 연계된 타 시스템의 data인지, 자신의 application내의 data인지 인지 할 필요가 없어야 한다.

# Endpoint
타 system의 application 으로 mesasge를 보내고 받는 역할을 하는 것을 Endpoint라 한다.  
이 Endpoint는 message 전송방법과 message에 변환을 통하여 application내의 service에서는 mesage가 외부로 부터 온 것인지 내부로 부터 온것인지 알 필요가 없게 해야 한다.  
##Endpoint 역할에 따른 분류
- consumer Endpoint
  외부로 부터 message를 받는 Endpoint
- producer Endpoint
  message를 외부로 내보내는 Endpoint

# 시스템간 통합에 생각되는 문제들
A라는 시스템은 B, C, D 라는 시스템과 message정보를 주고 받기로 했다. 하지만 B,C,D 시스템과 A 시스템간의 message 전송방법과, message 유형이 다르다면 Endpoint 구조를 어떻게 가져가야 할까?  
B는 HTTP 의 XML, C는 Message Queue 의 JSON, D는 file 의 text Byte 이라면 각각의 전송계층에 대한 Endpoint를 구현 해야 하며 각각의 message 변환기를 가져야 할 겻이다.  
이를 좀 유연하게 하려면 Endpoint를 3단계의 layer로 나누어 다음과 같이 하면 추가적인 전송계층과 message유형에 따라 좀더 유연 하게 응대 할 수 있다.  
- 전송 프로토콜 처리 -> router(Mesasge 유형에 따라) -> Message 변환기  
하지만 이렇게 할때 단점은 layer 분리에 따른 좀더 많은 component가 필요하며 타 시스템간에 연결이 중앙집중형(Star형)이 아닌 서로간에 peer-to-peer시 좀더 복잡해지고 많은 component들이 필요 해진다.  이러할 때는 시스템간에 직접연결이 아닌 중앙에 공통영역을 만들어 각 시스템은 공통영역를 통해 시스템간의 통신을 하는 것도 한 방법이 될 듯 하다.

# akka-camel
EndPoint 구현시 전송계층과 메시지를 처리해야 하는데 전송계층에 따른 구현은 쉽지 않고, 이를 구현하려면 전송방법에 따른 많는 수고를 감수 해야 한다.  
apache camel 은 이런 부분을 이미 구현하여 제공함으로써 이를 이용하면 쉽게 EndPonit를 만들 수 있으며 akka-camel를 이용하면 akka에서 이런 apache camel를 사용할 수 가 있다.  
- 참고: 
  현재 akka 2.5.12를 사용하고 있는데 akka-camel이 이제 alpakka(akka-stream에 기반한 다양  
  한 endpoint제공)으로 인해 deprecated 되었다. 추후 이에 대해 알아봐야 겠다. alpakka에 대한 
  자세한 정보는 아래 link 참조.  
  ['alpakka'](https://developer.lightbend.com/docs/alpakka/current/){: .btn.btn-default target="_blank" }

## camel의 지원 전송계층
HTTP, SOAP, TCP, FTP, SMTP, JMS등이 있으며 apache camel에서 지원가능한 전송계층에 따르는 지원 component와 API가 있는데 이는 
['apache camel site'](http://camel.apache.org/components.html/){: .btn.btn-default target="_blank" } 를 참조하면 된다.  

## akka-camel 장점
akka-camel 을 사용하려면 Consumer Endpoint는 Consumer를, Producer Endpoint는 Producer를 사용하면 되며, 이들은 전송계층 구현을 감춰주기때문에 메시지와 interface messaage사이의 변환만 직접 구현 하면 된다.  
또한 어떠한 protocal를 사용할지를 실행 시점에 결정할 수도 있게 할 수 있다.

# akka-camel Consumer
외부 시스템으로 부터 메시지를 소비하는(수신하는) Consumer Endpoint를 작성해 보자.  
예전에는 Actor를 확장했는데 여기서는 akka.camel.Consumer를 확장해야 한다.  
{% highlight scala %}
class FileConsumerEndPoint(uri: String, processor: ActorRef) extends Consumer
{% endhighlight %}
위에 처럼 Consumer를 확장한 후 endpointUri를 override하고 Actor method인 receive 를 구현 하면 된다.  
이때 응답은 CamelMessage유형으로 받게 된다.  

## file protocal를 이용한 Consumer 구현
akka.camel.Consumer를 확장하고 endpointUri를 구현, 외부로 부터 받은 message(CamelMessage type)를 receive method에서 message를 변환한 후 다른 Actor에게 data를 전달 한다.  
{% highlight scala %}
class FileConsumerEndPoint(uri: String, processor: ActorRef) extends Consumer
  with ActorLogging {

  def endpointUri = uri

  def receive = {
    case msg: CamelMessage => {
      val content = msg.bodyAs[String]
      val xml = XML.loadString(content)
      val order = xml \\ "order"
      val customer = (order \\ "customerId").text
      val productId = (order \\ "productId").text
      val number = (order \\ "number").text.toInt

      processor ! Order(customer, productId, number)
    }
    case msg =>
      log.debug(s"######## fileConsumerEndPoint receive other message")
  }
}
{% endhighlight %}

## CamelExtension 
akka-camel 모듈은 CamelExtension를 통해 camel를 actor system에서 사용할 수 있게 한다. 
이는  actorSystem 당 한번 load되며, 이를 통해 camelContext(camel compoenet life cycle관리등)와 ProducerTemplate(message생성관여)등에 접근 할 수 도 있다.  
camel 확장을 얻는 방법은 다음과 같다.  
{% highlight scala %}
val camelExtension = CamelExtension(system)
{% endhighlight %}
주의해야 할 점은 CamelExtension은 내부 component들을 비동기로 구성한다는 것 이다.  
만약 내가 만든 EndPoint camel확장이 준비가 된 것을 감지 하려면 아래와 같이 해야 한다.  
{% highlight scala %}
val camelExtension = CamelExtension(system)
val activated: Future[ActorRef] = 
	camelExtension.activationFutureFor(myConsumerEndPoint)(timeout = 10 seconds, executor = system.dispatcher)
{% endhighlight %}

## Test 해보기
commons-io를 이용해서 xml file를 local file system에 저장하게 되면 위의 code의 FileConsumerEndPoint가 이를 받아게 된다.  
code는 아래와 같다.  
{% highlight scala %}
implicit val executionContext = system.dispatcher
      
val probe = TestProbe()
val camelUri = "file:/Users/sslee/temp/"
val consumer = 
	system.actorOf(Props(new FileConsumerEndPoint(camelUri, probe.ref)))

val camelExtension = CamelExtension(system)
val activated = camelExtension.activationFutureFor(consumer)
	(timeout = 10 seconds, executor = system.dispatcher)

val msg = Order("sslee", "Akka in Action", 10)
val xml = 
  <order>
    <customerId>{msg.customerId}</customerId>
    <productId>{msg.productId}</productId>
    <number>{msg.number}</number>
  </order>

val msgFile = new File("/Users/sslee/temp/","order-20180512.xml")
FileUtils.write(msgFile, xml.toString.replace("\n",""))

//test대상 Consumer의 확장 Endpoint가 비동기로 생성되므로 준비가 될때까지 기다린다.
Await.ready(activated, 5 seconds)

probe.expectMsg(msg)
{% endhighlight %}




[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
