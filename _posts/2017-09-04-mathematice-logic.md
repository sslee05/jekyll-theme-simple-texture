---
layout: post
title: "수학적 논리 명제"
description: "mathematice logic"
categories: [mathematice]
tags: [mathematice]
redirect_from:
  - /2017/09/04/
---

> 수학적 논리.
>
> 수학의 개념과 관련된 책을 보고 정리한 한 것임.

* Kramdown table of contents
{:toc .toc}

# 논리

### implications
 국어 사전에 보니 말이나 글에 어떠한 뜻이 있음
 추론,함축,예상된 결과,영향
 책에서는 함의라 칭하고 있네.
 수학의 핵심은 implications라는 어떤 명제에 대한 검증으로 구성

### implication 구성
 첫번째 부분은 명제 그 자체의 가정.
 세번재 부분은 또 다른 명제인 결론
 둘째는 첫번째의 가정과 결론을 이어주는 기호
{% highlight mathematice %}
 ex) p => q
{% endhighlight %}

위의 의미는 다음과 같다.
* p는 q를 함의(implications) 한다.
* 만일 p이면 q 이다.
* q는 p에서 추론 된다.
* p는 q가 되기위한 충분조건 이다.
* q는 p가 되기 위한 필요조건 이다.

예는 다음과 같다.
{% highlight mathematice %}
C는 까마귀이다 => C는 새이다.
{% endhighlight %}

### 역 명제
{% highlight mathematice %}
 ex) p => q 의 역명제 q => p
{% endhighlight %}
p => q 이 True 라고 해서 역명제가 True라 할 수 없다.

### 정리
타당하면서 어느 정도 수학적으로 중요한 의미를 갖는다고 알려진 논리적 함의들을 "정리"라 한다.
수학은 p가 참이라고 가정하면, 논리와 수학의 규칙들을 이용하여 q라는 명제를 추론하게 된다. 그렇게 함으로써  p => q 라는 하나의 정리를 증명하는 것이다.

### 복합명제
단순명제의 연결이다.
{% highlight scala %}
p => q 
p와 q는 단순명제 p => q 는 복합명제 이다.
ex) C는 까마귀다 => C는 새이다.(복합명제)
C는 까마귀다 (단순명제)
C는 새이다( 단순명제)
{% endhighlight %}

### 진리표
* p => q 의 진리표

	|-----------------+------------+-----------------|
    |        p        |     q      |     p => q      |
    |----------------:|-----------:|----------------:|
    |       T         |     T      |        T        |
    |     **T**       |   **F**    |      **F**      |
    |       F         |     T      |        T        |
    |       F         |     F      |        T        |
    |-----------------+------------+-----------------|


위의 표를 보면 q가 false일때만 p => q 는 거짓 임을 알 수 있다.

* ∧(and)

	|-----------------+------------+-----------------|
    |        p        |     q      |      p ∧ q      |
    |----------------:|-----------:|----------------:|
    |       T         |     T      |        T        |
    |       T         |     F      |        F        |
    |       F         |     T      |        F        |
    |       F         |     F      |        F        |
    |-----------------+------------+-----------------|

* ∨(or)

	|-----------------+------------+-----------------|
    |        p        |     q      |      p ∨ q      |
    |----------------:|-----------:|----------------:|
    |       T         |     T      |        T        |
    |       T         |     F      |        T        |
    |       F         |     T      |        T        |
    |       F         |     F      |        F        |
    |-----------------+------------+-----------------|




[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture