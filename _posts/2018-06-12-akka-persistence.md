---
layout: post
title: "akka Cluster"
description: "akka persistence"
categories: [scala-akka]
tags: [scala,스칼라,akka,아카,persistence,persistent]
redirect_from:
  - /2018/06/12/
---

> akka persistence.
>


* Kramdown table of contents
{:toc .toc}

# Akka 상태 저장
event 에 대한 상태가 있는 상태저장이 필요한 Actor 인 경우 Actor system 이 shutdown등 문제가 생겨 메모리에 있는 Actor의 상태를 잃어버릴 경우 영속성 저장 장치에 기록하여 저장하는 방법을 제공한다.

# Event Sourcing
영속성 저장장치에 event의 저장을 journal 형식으로 저장하고 복구시 이를 다시 읽어 들여 복구하는 방식
Jounal plugin 들이 있으며, test용인 levelDB journal plug-in 이 있고, ['jdbc용 journal plug-in'](https://github.com/dnvriend/akka-persistence-jdbc){: .btn.btn-default target="_blank" }들이 있다. 참고 하면 될 듯 하다.  

# 적용 방법
상태를 journal 형태로 기록 해야 하는 Actor 인 경우  Actor을 상속하는 것이 아닌 PersistentActor를 확장 해야 한다.  

이는 event message 처리하는 method와 복구하는 method 2개가 있다.  

## receiveCommand: Receive
일반 Actor의 receive method에 해당 한다.  특이점은 이곳에서 message 처리 logic을 바로 실행 하는 것이 아닌 persist라는 method를 이용하여 event를 저장하고 message를 처리 하게끔하는 방법이다.  
{% highlight scala %}
def persist[A](event: A)(handler: A ⇒ Unit): Unit
{% endhighlight %}
첫번째 parameter 목록에 있는 parametr 인자가 성공적으로 저장되면 handler 함수가 호출 되며 이 handler function에 기존과 같은 logic를 처리 하면 된다.  

아래 예는  Akka in Action 에 있는 예이다.
{% highlight scala %}
def receiveCommand: Receive = {
  case Add(value) => persist(Added(value))(updateState)
  case Subtract(value) => persist(Subtracted(value))(updateState)
  case Divide(value) => 
    if(value != 0) persist(Divided(value))(updateState)
    else log.warning(s"cant not divided zero value $value")
  case Multiply(value) => persist(Multiplied(value))(updateState)
  case GetResult => sender() ! state.result
  case Clear => persist(Reset)(updateState)
}

//logic
def updateState: Event => Unit = {
  case Reset => state = state.reset
  case Added(value) => state = state.add(value)
  case Subtracted(value) => state = state.subtract(value)
  case Divided(value) => state = state.divide(value)
  case Multiplied(value) => state = state.multiply(value)
}
{% endhighlight %}

## receiveRecover: Receive
복구시 호출되는 method
{% highlight scala %}
def receiveRecover: Receive = {
  case event: Event => updateState(event)
  case RecoveryCompleted => log.info("Calculator recovery completed") 
}
{% endhighlight %}
복구 완료시 RecoveryCompleted를 받게 된다

## persistenceId
event 저장시 해당 Actor event 식별 id  이다. 이를 이용하여 복구대상을 data를 식별 할 수 있게 된다.

# 설정




[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
