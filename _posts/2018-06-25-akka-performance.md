---
layout: post
title: "akka Performance"
description: "akka performance"
categories: [scala-akka]
tags: [scala,스칼라,akka,아카,성능,performance]
redirect_from:
  - /2018/06/25/
---

> akka cluster.
>


* Kramdown table of contents
{:toc .toc}

service에 있어 성능에 관한 일반적인 것들과 Akka actor system에서의 특징에 따른 성능에 관한 사항을 정리 해본다.

# 성능에 관한 일반적인 내용들
- 병목현상(bottleneck)
전체 system 의 성능을 한정시키는 부분 주로 block 인 부분에서 발생됨

- Throughput(스루풋)
일정한 시간동안 요청을 처리 할 수 있는 작업의 수

- Service 속도
일정한 시간동안 요청을 처리한 평균 작업수

- Service(서비스시간)
한 작업을 처리하는데 걸리는 시간

- Latancy(지연시간)
요청에 대한 응답 처리하는데 지연 혹은 대기 되는 시간

- Pareto Principle
20%의 문제를 해결하면 80%의 성능 향상을 가져 온다.

- disminishing return(수확체감)
두번째 개선에 따른 효과 크기는 처음 개선의 따른 효과 크기보다 작다.  이는 개선의 증가에 따른 시스템 자원의 효율의 극대로 인한 성능개선의 효과가 기울기가 완만해진다. 따라서 이때는 scale out를 고려 할 수 있다.

- Utilization (가동률)
어떠한 작업의 일을 처리함에 있어 그 것을 처리하는데 소요된 전체에 대한 비율  
예) 가동율 50% => 일을 하는데 50% 시간을 소비하고 50%는 유휴 시간  

# Akka Actor System에서 성능에 관한 고려 사항들
![latency]({{ baseurl }}/assets/images/akka-performance/akka-performance.png)  
scala out이 아닌 단일 node단위의 성능에 관한 사항은 위에 그림에서 보듯이 mailbox의 대기열시간, 서비스 시간, 가동율에 따른 thread pool 의 조절이 핵심이다.

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
