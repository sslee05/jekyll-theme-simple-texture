---
layout: post
title: "akka Cluster"
description: "akka cluster"
categories: [scala-akka]
tags: [scala,스칼라,akka,아카,Cluster,클러스터]
redirect_from:
  - /2018/06/09/
---

> akka cluster.
>


* Kramdown table of contents
{:toc .toc}

# Akk Cluster
node로 이루어져 있으며, akka-remote 위에 network를 통해 node간 gossip protocol를 주고 받으며 heath check 를 한다.  
## Cluster의 사용 목적
- cluster membership 를 통한 내고장성(fault-tolerant) 보장  
- routing 기능을 활용한 부하균등화 (load balancing)  
- 특정기능만을 분할하여 node에 담당하게 할 수 있는 node 분할(node partitioning)  

# Cluster Membership
같은 Cluster 에 있는 node 들의 ActorSystem의 이름은 같아야 한며, cluster에 있는 모든 node의 설정에는 반드시 모든 seed node의 정보가 있어야 한다.

## seed node
cluster에는 특수한 목적을 가지는 필수 node가 있는데 이를 seed node라 한다.  
이는 cluster를 설립하는 시작점 역할을 하는 node이며 1개 이상의 node를 seed로 설정 하면 seed node의 down이 발생해도 다른 seed node가 역할을 대행 할 수 있게 되어 내고장성을 유지 할 수 있다.  

seed node가 없는 cluster에 일반 node가 최초로 실행시 seed node가 생성될때 까지 대기 한 후 seed node가 cluster에 join시 seed node가 먼저 올라온 node를 join 시킨 후 정상 서비스가 이루어 진다.  

## cluster 가입
node가 합류요청을 보내면 seed node가 합류 결정을 내리며 요청시 JOINING상태가 되며 seed node 에서 합류 결정이 이루어 지면 UP 상태가 된다.  

## cluster 이탈
cluster에서의 이탈은 JVM down이거나, event를 통한 이탈이 있을 수 있는데 JVM down시 seed-node에서 이탈된 node의 unreachable을 알고 이를 Exit 결정을 내려 remove시킨다.  
exit를 거처 remove 되지 않은 상태인 unreachable일때 같은 node가 가입하려 하면 가입할 수 없다.  
event를 통한 명시적 이탈시 다음과 같이 코드 상에서 이탈을 할 수 있다.  
{% highlight scala %}
val cluster = Cluster(system)
cluster leave cluster.selfAddress
{% endhighlight %}

### auto down
Unreachable 상태가 Exiting를 거처 Removed 되지 않고 Unreachable상태 발생시 Down상태로 자동 설정하게 할 수 있는 option이 있다.  
하지만 이는 운영 환경에서는 절대 하면 안된다. 왜냐 하면 Full GC중이 거나 잠시 network문제 였다면 이 node는 기존 cluster 에 합류가 아닌 일명 island node 가 될 수 있기 때문이다.  
따라서 이는 Test시에만 설정 하자. 설정 방법은 아래와 같다.  
{% highlight scala %}
cluster.autop-down-unrachable-after = 20s
{% endhighlight %}

# cluster 설정
remote위에 실행되는 module이므로 remote설정과 cluster설정이 있어야 하며, 모든 node에는 seed node의 주소가 모두 설정 되어 있어야 한다.  
또 주의 할 점은 같은 cluster에 있는 node의 ActorSystem명은 반드시 같아야 한다.  
또한 role 정보가 있는데 이는 node유형을 선택할때 유용 하다. 예를 들면 routee를 원격지에 배포 할때 특정 node 유형 즉 role을 지정할 수 가 있다. 설정의 예는 다음과 같다.
{% highlight scala %}
akka {

  #provider에 Cluster용으로 선언 
  actor {
    provider = "akka.cluster.ClusterActorRefProvider"
  }

  #해당 node의 원격지 정보
  remote {
    enabled-transports = ["akka.remote.netty.tcp"]
    log-remote-lifecycle-events = off
    netty.tcp {
      hostname = "127.0.0.1"
      port = 2551
      port = ${PORT}
    }
  }

  #cluster설정으로 모든 seed node에 대한 정보가 있어야 한다.
  cluster {
    seed-nodes = [
    "akka.tcp://words@127.0.0.1:2551",
    "akka.tcp://words@127.0.0.1:2552",
    "akka.tcp://words@127.0.0.1:2553"
    ]

    #해당 node의 role 정보
    roles = ["seed"]

    #Cluster가 정상 cluster의 구성이라고 판단하는 설정
    # 야래의 예는 seed node가 1개 이상, master node가 1이상 , worker node가 2이상
    # 일때 정상(실행가능) cluster로 판단 한다.
    role {
      seed.min-nr-of-members = 1
      master.min-nr-of-members = 1
      worker.min-nr-of-members = 2
    }
  }
}
{% endhighlight %}

# Cluster membership event 구독
cluster의 Node의 상태 Event들을 구독, 구독 해제 할 수 있다. 
{% highlight scala %}
class ClusterDomainEventListener extends Actor with ActorLogging {
  
  override def postStop() = {
    //구독 해제 
    Cluster(context.system).unsubscribe(self)
    super.postStop()
  }
  
  override def preStart() {
    //ClusterDomainEvent에 대한 구독 시작 
    Cluster(context.system).subscribe(self,classOf[ClusterDomainEvent])
  }
  
  def receive = {
    case MemberUp(member) => log.info(s"#####$member UP")
    case MemberExited(member) => log.info(s"#####$member exited")
    case MemberRemoved(member,previousState) => 
      if(previousState == MemberStatus.Exiting)
        log.info(s"#####$member gracefully removed")
      else
        log.info(s"#####$member downed after unreachable removed")
        
    case UnreachableMember(member) => log.info(s"#####$member unreachable")
    case ReachableMember(member) => log.info(s"member reachable")
    case s: CurrentClusterState => log.info(s"cluster state: $s")
  }
  
}
{% endhighlight %}

![functor]({{ baseurl }}/assets/images/akka-stream/stream07.png)



[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
