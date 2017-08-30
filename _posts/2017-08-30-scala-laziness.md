---
layout: post
title: "A quick demo of Simple Texture theme's code highlighting features"
description: "A quick demo of Simple Texture theme's code highlighting features"
categories: [functional_language]
tags: [scala]
redirect_from:
  - /2017/08/30/
---

> scala laziness.

* Kramdown table of contents
{:toc .toc}

# laziness 의 궁극적인 목적은

This is a test for inline codeblocks like `C:/Ruby23-x64` or `SELECT  "offices".* FROM "offices" `

Here is a literal `` ` `` backtick.
And here is a Ruby code fragment `x = Class.new`{:.language-ruby}

# call-by-value & call-by-name

Scala 에서  parameter는 default가 call-by-value이다. 

이것의 의미는 parameter의 대상의 결과가 인자로 넘어 간다는 것이다. parameter type 이 function 이면 실행 결과가 Fucntion의 instance 를 받는 것 이므로 혼동하지 말자

Call-by-name은 계산의 결과가 아닌 계산의 식이 넘어 가고 평가는 실제로 계산의 실행 시점에 평가가 된다.

하지만 parameter를 call-by-value가 아닌 call-by-name으로 인자를 넘길 수 있는데  이러한 기법을 scala non-strictness 또는 laziness 라 한다.

def fn(a:Int,b:Int):Int = a + b

def clientFn:Int = fn(1+2,3+4) 의 전계은 1+2,3+4의 계산 결과인 3,7이 fn의 인자로 넘어 간다. 이것이 call-by-value
반면 계산식 자체인 1+2, 3+4 가 넘어 가서 fn 의 body 안에 a + b를 만나는 곳에서 a = 1+ 2 가 대입되어 평가 되고 b = 3+ 4가 대입되어 평가 되어서 최종 (1+2) + (3 + 4)  가 되는 것이 call-by-name이다.

Scala 에서 parameter를 call-by-name으로 할 수 있게 하는 방법은 2가지 표현이 있다.
a:() => A 인 명시적 thunk 표현식 ,a: => A 인 표현이 있다.

참고로 class parameter는 => A 표현이 안된다.
() => A 로 만 해야 함.
{% highlight scala %}
sealed case class Cons[+A](head:() => A,tail:() => Stream[A]) extends Stream[A]
{% endhighlight %}

# Simple codeblock with long lines

    function myFunction() {
        alert("Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.");
    }

# Language of Code Blocks

~~~ ruby
def what?
  42
end
~~~

# Highlighted

## External Gist

<script src="https://gist.github.com/yizeng/9b871ad619e6dcdcc0545cac3101f361.js"></script>

## Simple Highlight

{% highlight ruby %}
def foo
  puts 'foo'
end
{% endhighlight %}

## Highlight with long lines

{% highlight c# %}
public class Hello {
    public static void Main() {
        Console.WriteLine("Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.");
    }
}
{% endhighlight %}

## Highlight with line numbers and long lines

{% highlight javascript linenos=table %}
function myFunction() {
    alert("Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.");
}
{% endhighlight %}

[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture