---
layout: post
title: "akka Integration"
description: "akka Integration"
categories: [scala-akka]
tags: [scala,스칼라,akka,아카,Integration,EIP,아카시스템통합]
redirect_from:
  - /2018/05/12/
---

> akka Integration.
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
Endpoint 역할에 따른 분류를 하면 다음과 같다.  
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
  자세한 정보는 ['alpakka'](https://developer.lightbend.com/docs/alpakka/current/){: .btn.btn-default target="_blank" } 참조.  ====

## camel의 지원 전송계층
HTTP, SOAP, TCP, FTP, SMTP, JMS등이 있으며 apache camel에서 지원가능한 전송계층에 따르는 지원 component와 API가 있는데 이는 
['apache camel site'](http://camel.apache.org/components.html/){: .btn.btn-default target="_blank" } 를 참조하면 된다.  

## akka-camel 장점
akka-camel 을 사용하려면 Consumer Endpoint는 Consumer를, Producer Endpoint는 Producer를 사용하면 되며, 이들은 전송계층 구현을 감춰주기때문에 메시지와 interface messaage사이의 변환만 직접 구현 하면 된다.  
또한 어떠한 protocol를 사용할지를 실행 시점에 결정할 수도 있게 할 수 있다.

# akka-camel Consumer
외부 시스템으로 부터 메시지를 소비하는(수신하는) Consumer Endpoint를 작성해 보자.  
Actor를 확장하는 대신 여기서는 akka.camel.Consumer를 확장해야 한다.  
{% highlight scala %}
class FileConsumerEndPoint(uri: String, processor: ActorRef) extends Consumer
{% endhighlight %}
위에 처럼 Consumer를 확장한 후 endpointUri를 override하고 Actor method인 receive 를 구현 하면 된다.  
이때 응답은 CamelMessage유형으로 받게 된다.  

## file를 통한 전송계층 Consumer 구현
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
	camelExtension.activationFutureFor(myConsumerEndPoint)
{% endhighlight %}

## Test 해보기
commons-io를 이용해서 xml file를 local file system에 저장하게 되면 위의 code의 FileConsumerEndPoint가 이를 받아게 된다.  
code는 아래와 같다.  
{% highlight scala %}

//camelExtension.activationFutureFor에서 필요한 항목들
implicit val executionContext = system.dispatcher
implicit val timeout:Timeout = 10 seconds

val probe = TestProbe()
val camelUri = "file:/Users/sslee/temp/"
val consumer = system.actorOf(Props(
	new ConsumerEndPoint(camelUri, probe.ref)))

val camelExtension = CamelExtension(system)
val activated: Future[ActorRef] = camelExtension.activationFutureFor(consumer)


val msg = Order("sslee", "Akka in Action", 10)
val xml = 
  <order>
    <customerId>{msg.customerId}</customerId>
    <productId>{msg.productId}</productId>
    <number>{msg.number}</number>
  </order>


val msgFile = new File("/Users/sslee/temp/","order-20180512.xml")
FileUtils.write(msgFile, xml.toString.replace("\n",""))

Await.ready(activated, 5 seconds)

probe.expectMsg(msg)
{% endhighlight %}

## 통신 protocol 변경에 따른 영향도
위의 예제에서 file 에서 TCP 통신으로 변경을 한다면 어떻게 될까?  
akka-camel를 이용하면 간단하게 endpointUri 정보만 변경하면 된다.  
protocol 에 다른 설정 정보는 ['apache camel site'](http://camel.apache.org/components.html/){: .btn.btn-default target="_blank" } 를 참조한다.  
{% highlight scala %}
implicit val executionContext = system.dispatcher
implicit val timeout:Timeout = 10 seconds

val probe = TestProbe()
val camelUri = "mina2:tcp://localhost:8080?textline=true&sync=false"
val consumer = system.actorOf(Props(new ConsumerEndPoint(camelUri, probe.ref)))

val activated = CamelExtension(system).activationFutureFor(consumer)
Await.ready(activated, 5 seconds)

val msg = Order("sslee", "Akka in Action", 10)
val xml = 
  <order>
	<customerId>{msg.customerId}</customerId>
	<productId>{msg.productId}</productId>
	<number>{msg.number}</number>
  </order>

val xmlStr = xml.toString.replace("\n","")
val socket = new Socket("localhost",8080)
val outputWriter = new PrintWriter(socket.getOutputStream,true)
outputWriter.println(xmlStr)
outputWriter.flush()

probe.expectMsg(msg)

outputWriter.close()
{% endhighlight %}
위의 test code 를 보면 기존 Consumer EndPoint의 수정은 message 유형만 같다면 변경하지 않아도 된다.  

# akka-camel Producer
타 시스템의 서비스를 요청하기 위해 요청 message를 만들어(생산)해서 보내야 한다.  
이를 위해서는 akka.camel.Producer를 확장하면 된다.  
{% highlight scala %}
class ProducerEndPoint(uri: String) extends Producer
{% endhighlight %}
그리고 Actor abstract method인 receive는 Producer 에 구현이 되어 있다.  
따라서 endpointUri만 해당 하는 것으로 override 하면 된다.  
## 요청 message변환
요청을 보낼 message의 유형을 변환 해야 한다면 transformOutgoingMessage를 
override하면 된다.  
{% highlight scala %}
override protected def transformOutgoingMessage(message: Any): Any = ???
{% endhighlight %}
이 method는 message를 보내기 전에 호출 된다.  

## 응답 message변환
타 시스템의 서비스에 요청을 보내고 응답을 받아야 하는 경우와 받지 않아도 되는 경우가 있을 것이다.  
응답을 받지 않아도 된다면 다음의 설정을 받드시 override해야 한다. 그렇게 함으로써 응답을 기다리기 위한 resource를 허비하지 않게 된다.  
{% highlight scala %}
override def oneway = true
{% endhighlight %}
위의 설정 default값은 false로 되어 있어 응답을 기다리는 것이 기본으로 된다.  
응답을 받기로 했다면 응답은 CamelMessage 유형에 감싸여 받게 된다.  이때 해당 Application안의 service에서 받은 응답의 유형을 아마도 그대로 사용하지 않을 것이다.  
따라서 변환 작업을 해주어야 하는데 이때 Producer에 변환을 위한 hock method를 제공 해주며 이 method는 요청을 보낸던 Actor에게 송신하기 전에 호출 된다.  
{% highlight scala %}
override def transformResponse(message: Any): Any = ???
{% endhighlight %}
만약 타 시스템의 서비스에 대한 요청응답을 ProducerEndPoint가 받아서 요청을 응답했던 Actor에게 보내 주는 것이 아닌 다른 Actor에게 보내야 한다면 routeResponse라는 method를 override하면 된다. 이때 routeResponse method안에 명시적으로 transformResponse를 호출해야 변환 message를 target actor에게 보내 줄 수 있다.  

### producer 예제 
{% highlight scala %}
class ProducerEndPoint(uri: String) extends Producer with ActorLogging {
  def endpointUri = uri
  //요청에 대한 응답을 기다리겠다는 설정 , 응답을 기다리지 않는다면 true설정
  //true시 이는 기다리기 위한 resource를 사용하지 않는다.
  override def oneway = false

  //전송시 message 변환 작업이 필요시 override한다.
  //이는 message전송 직전에 호출 된다.
  override protected def transformOutgoingMessage(message: Any): Any = 
  	message match {
    case msg: Order => {
      val xml = 
        <order>
          <customerId>{msg.customerId}</customerId>
          <productId>{msg.productId}</productId>
          <number>{msg.number}</number>
        </order>

       xml.toString().replace("\n", "")
    }

    case other => 
	  log.debug(s"##### not supported message typ. message is $other")
  }

  //요청에 요청했던 송신자에게 전송하기 전에 호출된다.
  //이를 통해 응답을 Application안에서 사용하는 type으로 변환 한다.
  override def transformResponse(message: Any): Any = message match {
    case msg: CamelMessage => 
      try {
        log.debug(s"##### ProducerEndPoint receive message $msg")
        val content = msg.bodyAs[String]
        val xml = XML.loadString(content)
        val res = (xml \\ "confirm").text
        res
      }
      catch {
        case e: Exception =>
          s"TransformException ${e.getMessage}"
      }
  }

}
{% endhighlight %}

## test code
요청을 받아 서비스할 application쪽의 Consumer Endpointer를 선언하고, 요청하는 쪽의 Application쪽의  요청을 보낼 Producer Endpointer를 선언 한다.  
Order 요청 message를 Producer에 보내고 Consumer가 응답을 받고 결과를 다시 Proceduer쪽으로 송신 한다. Proceduer 는 송신 message를 받아 요청했던 actor에게 전달 한다. 
{% highlight scala %}
//activationFuturFor를 위한 implicit 선언 
implicit val ExecutionContext = system.dispatcher
implicit val timeout: Timeout = 10 seconds

"Producer EndPoint" must {
  "using TCP request and response callback" in {
    //결과 검증 probe
    val probe = TestProbe()
    //중앙 공통 interface 
    val camelUri = "mina2:tcp://localhost:8080?textline=true"

    //요청을 받아 서비스를 처리하는 system의 consumerEndPoint
    val consumer = system.actorOf(Props(
  	  new ResponseConsumerEndPoint(camelUri,probe.ref)))
    //서비스 요청을 할 system의 producer EndPoint
    val producer = system.actorOf(Props(new ProducerEndPoint(camelUri)))

    //camel component를 확장한다.
    val activatedSystem = CamelExtension(system)
    //consumer camel을 확장한 ActorRef
    val activatedCons: Future[ActorRef] =
  	  activatedSystem.activationFutureFor(consumer)
    //producer camel을 확장한 ACtorRef
    val activatedProd: Future[ActorRef] =
  	  activatedSystem.activationFutureFor(producer)

    val camels: Future[List[ActorRef]] = 
  	  Future.sequence(List(activatedCons, activatedProd))
    Await.result(camels,3 seconds)

    val requester = TestProbe()
    val msg = new Order("sslee", "Akka in Action", 10)
    requester.send(producer, msg)

    probe expectMsg msg
    requester expectMsg "OK"

    system.stop(consumer)
    system.stop(producer)
  }
}
{% endhighlight %}

# akka-http를 EndPoint로 사용하기  
camel은 다양한 전송 protocol endpoint를 제공해준다. 하지만 특정 protocol에 상세한 기능을 제공할 수 는 없다.  
예를 들어 HTTP protocol에서 camel를 이용하여 HTTP interface를 구현할 수도 있지만 좀더 많는 기능이나 camel 만으로 안되는 경우 akka-http 를 이용 할 수 있다.  

## akka-http REST endpoint 
http route 를 만들어 요청에 대한 응답을 처리하고 결과를 반환하는 하는 endpoint를 구성한다. 이는 요청에 대하여 해당 Application 의 Actor가 처리리 할 수 있게 message를 변환한다.  
또한 client가 key에 대한 조회 서비스를 요청하면 이때는 producer endpoint 역할로써 해당 Applicatoin의 처리 결과에 대하여 client에게 보내질 형식으로 변한를 거친후 client에게 message를 전달 하게 된다.  
### route 생성
이곳에서 route를 생성하고 메시지 변환을 거처 서비스를 처리할 Actor에게 요청을 처리하게 하거나, 요청응답에 대하여 서비스를 처리할 Actor에게 요청을 보낸수 결과를 client에게 전송 한다.  
{% highlight scala %}
trait OrderService {

  implicit def timeout: Timeout
  implicit def executionContext: ExecutionContext

  //서비스를 처리할 Actor
  //이를 추상으로 하게 함으로써 
  //remote proxy ActorRef, local ActorRef, testProbe 과 
  //route와의 관심사를 분리 시킨다.
  val orderProcessor: ActorRef

  def routes = getOrder ~ postOrders

  def getOrder = get {
    pathPrefix("orders" / IntNumber) { id =>
      onSuccess(orderProcessor ask TrackingId(id) ) {
        case result: TrackingOrder => 
          complete(
            <statusResponse>
			  <id>{result.orderId}</id>
			  <status>{result.status}</status>
            </statusResponse>
          )
        case result: NoSuchOrder =>
          complete(StatusCodes.NotFound)
      }
    }
  }

  def postOrders = post {
    path("orders") {
      entity(as[NodeSeq]) { xml =>
        val order = toOrder(xml)
        onSuccess(orderProcessor ask order) {
          case result: TrackingOrder => 
            complete(
              <confirm>
                <id>{result.orderId}</id>
                <status>{result.status}</status>
              </confirm>    
            )
          case result => complete(StatusCodes.BadRequest)
        }
      }
    }
  }
}
{% endhighlight %}
route과 route에서 사용할 Actor를 분리하게 함으로써 local, remote, test 환경에 따른 영향도를 없앤다.  
onSuccess는 RouteDirectives 의 method이며 OnSuccessMagnet를 parameter로 받는다.  
OnSuccessMangent 는 OnSuccessMagnet object에 apply 적용함수를 보면 Future를 받는다.
{% highlight scala %}
implicit def apply[T](future: ⇒ Future[T])(implicit tupler: Tupler[T])
{% endhighlight %}
xml의 형변환에 사용되는 ToEntityMarshaller와 FromEntityMarshaller이 있어야 하며 이는 akka.http.scaladsl.marshallers.xml.ScalaXmlSupport._를 import 함으로써 이들에 대한 암시적 사용이 가능하다.  
akka-http 에 대한 것은 따로 blog를 작성할 것 이다.  

어찌되었든 여기서 중요한 것은 akka-http 의 route 또한 하나의 HTTP REST endpoint와 같은 역할을 하는 것을 볼 수 있다.(전송계층역할 + message 변환 역할)  

# akka-http testKit
이를 사용하면 scalatest와 별도로 build.sbt에 dependency를 추가 해야 한다.
이를 이용하면 route를 테스할 수 있는 DSL(Domain special language)를 사용할 수 있다.
예는 아래와 같다.
{% highlight scala %}
//ScalatestRouteTest를 mix-in 
class OrderServiceTest extends WordSpecLike 
  with MustMatchers with OrderService with ScalatestRouteTest {

  implicit val executionContext = system.dispatcher
  implicit val timeout = akka.util.Timeout(3 seconds)

  val orderProcessor = system.actorOf(Props(new OrderProcessor))

  "The order processor" must {
    "return not found if trackingId not found" in {
      Get("/orders/1") ~> routes ~> check {
        status mustEqual StatusCodes.NotFound
      }
    }

    "return tracking id for order that was post" in {
      val xml = 
        <order>
		  <customerId>sslee</customerId>
          <productId>akka in action</productId>
          <number>10</number>
        </order>

      Post("/orders",xml) ~> routes ~> check {
        status mustEqual StatusCodes.OK

        val responseXml = responseAs[NodeSeq]
        val id = (responseXml \\ "id").text.toInt
        val orderStatus = (responseXml \\ "status").text
        id mustEqual 1 
        orderStatus mustEqual "created"
      }

      Get("/orders/1") ~> routes ~> check {
        status mustEqual StatusCodes.OK

        val responseXml = responseAs[NodeSeq]
        val id = (responseXml \\ "id").text.toInt
        val responseStatus = (responseXml \\ "status").text

        id mustEqual 1
        responseStatus mustEqual "processing"
      }
    }
  }

}
{% endhighlight %}



[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
