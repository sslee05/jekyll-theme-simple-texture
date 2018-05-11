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
아래는 FSM의 code 이부분을 나타낸다.  
{% highlight scala %}

//천이 부분을 기술
type StateFunction = scala.PartialFunction[Event, State]
final def when(stateName: S, stateTimeout: FiniteDuration = null)(stateFunction: StateFunction): Unit

//천이시 진입동작을 기술
type TransitionHandler = PartialFunction[(S, S), Unit]
final def onTransition(transitionHandler: TransitionHandler): Unit
{% endhighlight %}
위의 code에 when method  1번째 parameter 목록를 보면 특정 상태일때와 2번째 parameter목록의 StateFunction 즉 부분함수를 기제 한다. StateFunction의 타입파라미터를 보면 Event, State로 Event를 받아을때 천이할 State을 반환함을 추측 할 수 있다.  
또 onTransition method 의 parameter또한 부분함수로 두 가지의 상태 천이전 상태와 천이후의 상태를 Tuple 형태로 받음을 추측 할 수 있다.  


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
