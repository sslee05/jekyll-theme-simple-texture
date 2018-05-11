---
layout: post
title: "akka FSM"
description: "akka FSM"
categories: [scala-akka]
tags: [scala,스칼라,akka,아카,FSM,유한 상태 머신]
redirect_from:
  - /2018/05/10/
---

> akka FSM.
>


* Kramdown table of contents
{:toc .toc}

# Akka FSM
Akka 는 유한상태기계(Finite State Machine) 모델를 지원한는 akka.actor.FSM 를 지원한다.  
Akka에서 상태변경을 간당하게 become/unbecome를 통해 했다.  
또 다른 방법은 akka.actor.FSM를 이용하는 것으로 FSM에 대하여 이야기 한다.  

# 유한상태기계 모델
유한상태기계 모델은 유한한 상태가 있고 시스템은 동시에 여러 상태가 될 수 없으며, 오직 한 특정 상태에만 존재 할 수 있다.  
상태는 어떤 이벤트(사건)으로 상태가 변경이 일어 나는데 이를 천이(transition)이라 한다.  
그리고 천이시 다음 상태로의 진입동작이 있다.

# Akka에서의 유한상태모델
akka 에서 유한상태모델을 지원하는 FSM를 가지고 있으며 유한상태기계 모델에 맞게 FSM은 다음과 같이 기술할 수 있는 방법을 제공 한다.  
FSM의 작성 방법은 다음과 같다.  
1. 유한한 상태를 정의 하고 특정 상태에서 이벤트를 받아 천이하는 부분을 기술 한다.  
2. 천이시 진입동작을 기술한다.

# FSM 들여다 보기
scala.actor.FSM에 살펴 볼 항목들은 when, onTransition 부분이다.  
아래는 FSM의 일부 code 부분을 나타낸다.  
{% highlight scala %}

//천이 부분을 기술
type StateFunction = scala.PartialFunction[Event, State]
final def when(stateName: S, stateTimeout: FiniteDuration = null)(stateFunction: StateFunction): Unit

//천이시 진입동작을 기술
type TransitionHandler = PartialFunction[(S, S), Unit]
final def onTransition(transitionHandler: TransitionHandler): Unit
{% endhighlight %}
위의 code에 when method  1번째 parameter 목록를 보면 특정 상태일때와 2번째 parameter목록의 StateFunction 즉 부분함수를 기제 한다. StateFunction의 타입파라미터를 보면 Event, State로 Event를 받아을때 천이할 State을 반환함을 추측 할 수 있다.  
또 onTransition method 의 parameter또한 부분함수로 두 가지의 상태, 천이전 상태와 천이후의 상태를 Tuple 형태로 받음을 추측 할 수 있다.  

# FSM 사용해 보기
## FSM trait 2개의 type parameter
FSM를 적용시 상태가 필요한 Actor에 FSM를 mix-in 해야 한다. 이때 FSM에는 2개의 type parameter를 받는데 이는 다음의 예제를 보자.
{% highlight scala %}
class BookStore(publisher: ActorRef) extends Actor with FSM[State, StateData]
{% endhighlight %}
위의 code에서 FSM[State, StateData]를 보면 State는 BookStore의 상태들의 super type를 뜻한다. StateData는 FSM이 추적할 상태 데이터 타입이다. 예제에서의 State와 StateData는 다음과 같다.  
{% highlight scala %}
//State
sealed trait State
case object WaitRequest extends State//요청을 기다리는 상태
case object ProcessRequest extends State//요청에 대한 결과를 처리하는 상태
case object WaitPublisher extends State//다른 Actor에 요청한 응답을 기다리는 상태
case object SoldOut extends State//판매 완료되어 재고가 없는 상태

//StateData 이는 FSM이 추적할 상태 데이터 타입
case class StateData(nrBookInStore: Int, penddingRequest: Seq[RequestBook])
{% endhighlight %}
StateData는 when method의 2번째 parameter 목록의 StateFunction의 Event유형에 stateData field에 담겨 진다. 이는 아래 천이부분에서 알 수 있다.  

## 상태천이(state transition)
상태천이 부분의 구현은 머신이 가질 수 있는 모든상태에 대하여 기제 한다. 이때  FSM의 when method를 이용하여 구현 한다.  
{% highlight scala %}
when(ProcessRequest) {
  //ProcessRequest상태에서 Done이라는 event가 발생시 WaitRequest로 천이 된다.
  case Event(Done, stateData: StateData) =>
    val newStateData = 
    stateData.copy(nrBookInStore = stateData.nrBookInStore - 1, 
          penddingRequest = stateData.penddingRequest.tail)
    goto(WaitRequest) using newStateData
    //stay 
}
when(WaitRequest) {
  case Event(....)
  //...
}
//중략...
{% endhighlight %}
위의 코드 정의는 ProcessRequest상태에서 Done이라는 event가 발생시 WaitRequest로 천이 됨을 기술 한 것 이다.  
위의 when method의 1번째 parameter목록은 상태를, 두번째 parameter 목록에는 StateFunction을 기술 한다. 이때 Event는 Event(event: bookStore예제에서의 이벤트,stateData: FSM이 추적할 상태 데이터 타입 예기에서는 위의 case class인 StateData이다. ) 이다.  
이는 FSM trait mix-in시 type parameter로 선언 했던 항목이다.  
goto(WaitRequest) using newStateData 부분은 goto(천이할 상태)이고, using은 FSM에서 추적할 StateData를 넘겨 주는 부분이다. 그러면 다음 상태에서 Event의 stateData항목에서 bind된다. 다음 상태로의 천이를 위해 goto를 사용 했는데 다음 상태로 천이 하지 않고 현 시점의 상태로 머무르고 싶다면 stay 를 선언하면 된다.  

이부분은 상태마다 예상하는 이벤트를 정의 하고 그 이벤트가 발생할 때 어디로 천이할지를 지정한다.

## 상태진입 행동
상태 진입에 대한 행동 기술은 onTransition의 파라메터인 부분함수를 구혆면 된다.  
이는 when에서 다음 상태로 천이시 천이전에 행위가 일어나고 다음 상태로 천이가 일어 난다.  
주의 할 점은 when에서 현재 상태에 머무르는 stay를 선언시 이 진입부분은 일어 나지 않는다.  
{% highlight scala %}
onTransition {
  case es -> WaitRequest =>
    log.debug(s"#####onTransition from $es to WaitRequest")
    if(!nextStateData.penddingRequest.isEmpty)
      self ! PenddingRequests

  case es -> WaitPublisher =>
    log.debug(s"#####onTransition from $es to WaitPublisher")
    publisher ! RequestPublisher
{% endhighlight %}
es -> WaitRequest는 Tuple이며 es는 천이전 상태, WaitRequest는 천이 후 상태를 나타낸다.  
따라서 위의 case es -> WaitRequest는 전 상태가 어떤 상태였던 WaitRequest로 천이시 진입해위를 표현 한다.  
nextStateData를 통해 when에서 다음 상태로 천이시 StateData를 넘긴 정보를 얻를 수 있다.  
이전 StateData는 stateData라는 변수로 얻을 수 있다.  

# whenUnhandled
whenUnHandled는 상태함수가 이벤트를 처리하지 않는 경우에 호출 된다. 따라서 when 에 기제된 상태마다 상태를 천이시키는 Event정보에 case에 일치 되지 않을 경우에 호출됨.  
{% highlight scala %}

when(WaitRequest) {
  case Event(event: RequestBook, stateData: StateData) =>...
  case Event(PenddingRequests, stateData: StateData) => ...
}

when(WaitPublisher,stateTimeout = 4 seconds) {
  case Event(event: BookSupply, stateData: StateData) => ...
  case Event(BookSoldOut, stateData: StateData) =>...
  case Event(StateTimeout, _) => ...
}

when(ProcessRequest) {
  case Event(Done, stateData: StateData) => ...
}

when(SoldOut) {
  case Event(Done, stateData: StateData) => ...
}

whenUnhandled {
  case Event(event: RequestBook, stateData: StateData) =>
    log.debug(s"whenUnhandled event $event stateData $stateData")
    stay using stateData.copy(
      penddingRequest = stateData.penddingRequest :+ event)

  case Event(e, s) =>
    log.debug(s" skip event event is $e stateData $s")
    stay
}
{% endhighlight %}
위의 경우에는 whenUnhandled가 호출 될때는 WaitPublisher,ProcessRequest,
ProcessRequest,SoldOut 상태에 있을 경우 RequestBook 이벤트 발생시 whenUnhandled의 case  Event(event: REquestBook ... 절이 호출 될 것 이다.  

# FSM shutdown hock
FSM가 종료시 hock method를 제공하는데 이는 다음과 같다.  
{% highlight scala %}
onTermination {
  //일반적인 종료, FSM 내부에서 stop()을 호출 한 경우 
  case StopEvent(FSM.Normal, state, data) => 
    log.debug(s"#####stopEvent Normal state:$state data:$data")
  //ㄹ느 이 중단됐을 때, FSM외부에서 system.stop(fsm)를 호출 한 경우 
  case StopEvent(FSM.Shutdown, state, data) =>
    log.debug(s"#####stopEvent Shutdown state:$state data:$data")
  // system 실패로 FSM이 종료 되었을 때
  case StopEvent(FSM.Failure(cause), state, data) =>
    log.debug(s"#####stopEvent Failure $cause state:$state data:$data")
}
{% endhighlight %}

# FSM 의 초기화 
FSM가 시작시 초기 상태를 지정해야 하는데 이는 startWith를 이용하고, 시작시 initialize로 초기화 한다.
{% highlight scala %}
startWith(WaitRequest, new StateData(0, Seq()))
initialize
{% endhighlight %}

따라서 FSM 의 사용은 다음과 같다.
{% highlight scala %}
class BookStore(publisher: ActorRef) extends Actor with FSM[State, StateData] {
  startWith(WaitRequest, new StateData(0, Seq()))
  when ...
  when ...
  when ...
  whenUnHandled ... //option 사항 
  onTransition ...
  initialize
  onTermination ... //option 사항 
}
{% endhighlight %}

# FSM test
상태의 천이에 대한 이벤트를 구독하여 예상대로 이벤트 천이가 되었는지 알아 보려면 해당 FSM에 SubscribeTransitionCallBack(monitor: ActorRef)의 message를 보내야 한다.
그리고 예상되는 상태를 기술 하면 된다.  
구독에 대한 응답은 FSM object에 기술 되어있다. 코드는 아래와 같다.  
{% highlight scala %}
/**
   * Message type which is sent directly to the subscribed actor in
   * [[akka.actor.FSM.SubscribeTransitionCallBack]] before sending any
   * [[akka.actor.FSM.Transition]] messages.
   */
  final case class CurrentState[S](fsmRef: ActorRef, state: S)

  /**
   * Message type which is used to communicate transitions between states to
   * all subscribed listeners (use [[akka.actor.FSM.SubscribeTransitionCallBack]]).
   */
  final case class Transition[S](fsmRef: ActorRef, from: S, to: S)
{% endhighlight %}

test하는 코드 예제는 아래와 같다.  
{% highlight scala %}
"BookStoreApp" must {

  "RequestBook state" in {
    val publisher = system.actorOf(Props(new Publisher(2,2)))
    val monitorProbe = TestProbe()
    val requestProbe = TestProbe()

    val bookStore = system.actorOf(Props(new BookStore(publisher)))
    //구독 신청  
    bookStore ! SubscribeTransitionCallBack(monitorProbe.ref)
    //초기 상태 응답예상 
    monitorProbe expectMsg(CurrentState(bookStore, WaitRequest))

    bookStore ! RequestBook(1, requestProbe.ref)

    //천이 상태 응답예상 
    monitorProbe expectMsg (new Transition(bookStore, WaitRequest,
  	  WaitPublisher))
    monitorProbe expectMsg (new Transition(bookStore, WaitPublisher,
  	  ProcessRequest))
    monitorProbe expectMsg (new Transition(bookStore, ProcessRequest,
  	  WaitRequest))
    requestProbe expectMsg BookReply(1,Right(1))
  }
}
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
