---
layout: post
title: "akka message channel"
description: "akka message channel"
categories: [scala-akka]
tags: [scala,스칼라,akka,아카,channel,채널]
redirect_from:
  - /2018/05/02/
---

> akka Channel.
>


* Kramdown table of contents
{:toc .toc}

# Message channel
Actor간의 message 전달은 message channel을 통해 이루어 진다.  
그런 channel 유형에 대표적인 두가지 channel인 점대점 채널(point-to-point)과 발행-구독(publish-subscribe-channel)을 둘 수 있다.  

# 점대점 채널 (Point-to-Point channel)
대부분의 ActorRef의 기본 channel이며, 한 번에 한 수신자만이 메시지를 받을 수 있다. 심지어 router라 하더라도 routee에 어떤 message에 대해서 처리는 한 routee만이 한다. 또한 message 처리 순서도 보장 된다.  

### 발행-구독 채널 (Publish-Subscribe channel)
이름에서 알수 있듯이 발신자에 대한 message를 여러 수신자가 수신한다는 것이고 발행한 message의 처리를 한 수신자가 하는 것이 아닌 여러 수신자가 처리를 한다.  
신문을 구독하는 것과 같다.  
그런데 점대점 channel 에서는 송신자가 message를 어디로 보낼지를 알고 있지만, 발생구독에서는 송신자는 누가 message에 관심이 있는지 알지 못한다. 수신자는 해당 구독할 정보를 channel에 등록해야 한다.  
또한 수신자는 runtime 시점에 구독을 할 수도, 해지 할 수도 있다.  

# akka EventStream
모든  ActorSystem에는 EventStream 을 하나씩 가지고 있다.  
수신자들은 이 EventStream을 통해 특정 message수신을 구독 할 수 있으며, EventStream은 syste.eventStream을 통해 접근 할 수 있다.  
또한 구독자는 동시에 여러 Message유형을 구독할 수 있다.  
발행 또한 EventStream을 통해 publish라는 method로 발행할 수 있다.  
{% highlight scala %}
//subscribeActor가 MyMessage 유형의 Message를 구독 신청
system.eventStream.subscribe(
	subscribeActorRef,
    classOf[MyMessage]
)

system.eventStream.subscribe(
	subscribeActorRef,
    classOf[MyMessage2]
)

//subscribeActor가 MyMessage 유형에 대한 구독을 취소
system.eventStream.unsubscribe(
	subscribeActorRef,
    classOf[MyMessage]
)

//발행자가 발행 
system.eventStream.publish(msg)
{% endhighlight %}

ActorLogging은 eventStream을 통해 event를 구독 한다.

# EventBus
akka에는 EventBus라는 trait가 있다. 이를 구현하면 custom 발행구독 채널을 만들 수 있다.  
EventBus에는 3가지 추상 type이 존재 한다. 아래 EventBus code를 보자   
{% highlight scala %}
trait EventBus {
  type Event //발행될 event type EventStream에서는 AnyRef
  type Classifier // 구독자를 분류할때 사용하는 기준 EventStream에서는 message의 type
  type Subscriber // 구독자 EventStream에서는 ActorRef

  //#event-bus-api
  /**
   * Attempts to register the subscriber to the specified Classifier
   * @return true if successful and false if not (because it was already
   *   subscribed to that Classifier, or otherwise)
   */
  def subscribe(subscriber: Subscriber, to: Classifier): Boolean

  /**
   * Attempts to deregister the subscriber from the specified Classifier
   * @return true if successful and false if not (because it wasn't subscribed
   *   to that Classifier, or otherwise)
   */
  def unsubscribe(subscriber: Subscriber, from: Classifier): Boolean

  /**
   * Attempts to deregister the subscriber from all Classifiers it may be subscribed to
   */
  def unsubscribe(subscriber: Subscriber): Unit

  /**
   * Publishes the specified Event to this bus
   */
  def publish(event: Event): Unit
  //#event-bus-api
}
{% endhighlight %}
위에 코드를 보면 3개의 추상 type을 구현하고 추상 method들을 구현하면 된다.  
- Event: event bus에 발행할 event type이다. 모든 ActorSystem 에 하나씩 있는 EventStrea의 Event는 AnyRef 으로 되어 있다.
- Classifier: 구독자자를 분류할때 사용하는 기준이다. EventStream에서는 Class[_]으로 되어있다
- Subscriber: 구독자 EventStream에서는 ActorRef

spring framework 에도 그러하듯 대게 interface에 해당하는 기본적인 구현체들을 제공한다. akka도 EventBus trait를 mixin 한 여러 trait가 존재 한다.  
따라서 위에 3개의 추상 type를 구현하고 나머지 추상 method들은 상황에 맞게 akka에서 제공하는 trait를 mixin해서 사용하면 된다.  

# akka 제공 Classification
## LookupClassification
구독자들을 집합으로 구분하고, abstract method 인 classify를 통해 구독자를 분별해서 발행 한다.  
말 보다는 아래 코드를 봐야 알기 쉽다.  
{% highlight scala %}
case class News(topic: String, description: String)

class MessageBusViaLookupClassification extends EventBus 
  with LookupClassification with ActorEventBus {
  
  type Event = News
  type Classifier = String
  //이는 ActorEventBus를 mix-in 하면 구현 ActorRef로 구현 되어 있다.
  //따라서 여기처럼 같은 type이면 구현 할 필요가 없다.
  type Subscribe = ActorRef
  
  //LookupClassification 를 mix-in 했다면 반드시 구현 해야 한다.
  //인덱스 데이터 구조의 초기 크기를 결정
  def mapSize = 128
  
  //LookupClassification의 Index 집합에 구독자를 분류해서 넣을때 호출된다.
  //`java.lang.Comparable.compare`
  //LookupClassification에는 abstract로 되어있지만 
  //ActorEventBus에 이미 아래와 같이 구현 되어있다.
  //따라서 여기처럼 같은 로직이면 구현 할 필요가 없다.
  override protected def compareSubscribers(a: Subscriber,b: Subscriber):Int = 
  	a compareTo b

  //event 시 구독자를 선별하기 위해 publish에서 사용된다.
  protected def classify(event: Event) = 
    event.topic
  
  protected def publish(event: Event, subscriber: Subscriber): Unit = 
         subscriber ! event.description
}
{% endhighlight %}
LookupClassifier의 내부에는 cake pattern으로 EventBus 를 선언하고 있다. 따라서 반드시 EventBus mix-in해야 한다.  
우선 위에는 LookupClassifier를 사용한다면 반드시 구현해야 하는 type 및 method들이다.  
그런데 type Subscribe는 ActorEventBus에 이미 ActorRef로 선언 되어있기 때문에 ActorEventBus를 mix-in 했다면 또 구현할 필요는 없다.  
그런 것이 또 있는데 compareSubscribers method역시 ActorEventBus에 똑 같이 구현되어 있다.  
LookupClassification에 구독자를 등록하는 method가 아래와 같이 되어 있다. 
{% highlight scala %}
protected final val subscribers = 
  new Index[Classifier, Subscriber](mapSize(), new Comparator[Subscriber] {
    def compare(a: Subscriber, b: Subscriber): Int = compareSubscribers(a, b)
  })
{% endhighlight %}
위의 코드를 보면 구독자들의 집합을 만들기 위해 집합의 초기화 크기인  mapSize와 compareSubscribers를 사용함을 알 수 있다.  
또한 발생시 classify의 결과에 해당하는 구독자들에게 event를 발행 한다.  

test source 는 아래와 같다.
{% highlight scala %}
val musicSubscriber = TestProbe()
  val recipeSubscriber = TestProbe()

  val eventBus = new MessageBusViaLookupClassification()
  eventBus.subscribe(musicSubscriber.ref, "musicNews")
  eventBus.subscribe(musicSubscriber.ref, "recipe")
  eventBus.subscribe(recipeSubscriber.ref, "recipe")

  eventBus publish News("musicNews",
  	"Jeo Satriani music festival 2018.05.02 pm 07:00")

  musicSubscriber.expectMsg("Jeo Satriani music festival 2018.05.02 pm 07:00")
  recipeSubscriber.expectNoMessage(3 seconds)

  eventBus publish News("recipe","Kimchi helthy food and delicious")

  musicSubscriber.expectMsg("Kimchi helthy food and delicious")
  recipeSubscriber.expectMsg("Kimchi helthy food and delicious")
{% endhighlight %}


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
