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
node로 이루어져 있으며, akka-remote 위에 network를 listen 하며, node간 서로간에 gossip protocol를 주고 받으며 heath check 를 한다.  
## Cluster의 사용 목적
- cluster membership 를 통한 내고장성(fault-tolerant) 보장  
- routing 기능을 활용한 부하균등화 (load balancing)  
- 특정기능만을 분할하여 node에 담당하게 할 수 있는 node 분할(node partitioning)  

# Cluster Membership


![functor]({{ baseurl }}/assets/images/akka-stream/stream07.png)



[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
