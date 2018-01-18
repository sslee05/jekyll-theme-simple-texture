---
layout: post
title: "scala Traversable Functor"
description: "Traversable Functor"
categories: [배움]
tags: [scala]
redirect_from:
  - /2017/09/08/
---

> scala Traversable Functor


* Kramdown table of contents
{:toc .toc}

# 순회가능 형식생성자의 공통 조합기 추상화 하기
List, Map등과 같은 형식생성자들의 자료구조와 함수를 받고 자료구조에 담긴 자료에 함수를 적용하여 원래의 구조를 보존의 기능을 추상화 한 Functor가 Traversable functor이다.  

우선 기존의 traverse, 혹은 sequence 함수들을 보면 다음과 같았다.
{% highlight scala %}
def traverse[F[_],A,B](xs: List[A])(f: A => F[B]): F[List[B]] = 
  xs.foldRight[F[List[A]]](unit(List())((a,b) => map2(f(a),b)((a1,b1) => a1::b1))
def sequence[F[_],A](xs: List[A]): F[List[A]] = 
  xs.foldRight[F[List[A]]](unit(List()))((a,b) => map2(a,b)(a1,b1) => a1 :: b1))
{% endhighlight %}

이를 Map에도 적용해보자.
{% highlight scala %}
def sequenceMap[K, V](xs: Map[K, F[V]]): F[Map[K, V]] =
  xs.foldLeft[F[Map[K, V]]](unit(Map.empty)) {
    case (b, (k, v)) => map2(b, v)((b1, v1) => b1 + (k -> v1))
}
{% endhighlight %}
위의 내용을 보면 구현형태가 비슷하다. 다만 해당 자료구조에 따라 달라진다.  
이들의 공통점을 추상화 하면 traversable functor라는 것을 만날 수 있다.

# traversable Functor의 추상화
{% highlight scala %}
trait Traverse[F[_]] {
  def traverse[G[_]: Applicative, A, B](xs: F[A])(f: A => G[B]): G[F[B]]
  def sequence[G[_]: Applicative, A](xs: F[G[A]]): G[F[A]] = 
    traverse(xs)(a => a)
}
{% endhighlight %}

자료구조와 함수를 받고 자료구조에 담긴 원소들을 함수에 적용해서 결과를 도출하는 면에서 fold 연산과 비슷하지만, 이는 결과가 원래의 자료구조를 보존하는 하지만 foldMap은 구조를 페기하고 Monoid의 이항연산으로 대처된다.(아래의 Foldable의 foldMap code 참조)
{% highlight scala %}
def foldMap[A,B](xs: F[A])(f: A => B)(m: Monoid[B]): B = 
  foldRight(xs)(m.zero)((a,b) => m.op(f(a),b))
{% endhighlight %}

# Traversable Functor & Functor
traversable functor가 Functor 이게 하려면 map을 traverse로 구현할 수 있어야 한다.  
이는 값만 감싸는 일명 IdMonad(monad는 Applicative Functor이므로)를 가지고 할 수 있다.

{% highlight scala %}
// 이렇게 implicit로 Applicative functor의 unit과 map2를 이용하여 변환
val listTraverse = new Traverse[List] {
  def traverse[G[_], A, B](xs: List[A])(f: A => G[B])(implicit G: Applicative[G]): G[List[B]] =
    xs.foldRight[G[List[B]]](G.unit(List()))((a, b) => G.map2(f(a), b)((a1, b1) => a1 :: b1))
}
{% endhighlight %}
위의 code처럼 Applicative functor의 unit 과 map2를 가지고 traverse를 구현하므로 아래처럼 IdMonad(Applicative functor)를 이용하면 됨.

{% highlight scala %}
trait Moand[F[_]] extends Applicative {
 ...
}

trait Traverse[F[_]] extends Functor[F] {
  def traverse[G[_]: Applicative, A, B](xs: F[A])(f: A => G[B]): G[F[B]]
  def sequence[G[_]: Applicative, A](xs: F[G[A]]): G[F[A]] =
    traverse(xs)(a => a)

  type Id[A] = A
  val idMonad = new Monad[Id] {
    def unit[A](a: => A) = a
    override def flatMap[A, B](ma: Id[A])(f: A => Id[B]): Id[B] = f(ma)
  }

  def map[A, B](ma: F[A])(f: A => B): F[B] =
    traverse[Id, A, B](ma)(f)(idMonad)
}
{% endhighlight %}

# Traversable Functor & Foldable
이전에 monoid 에서 접기자료구조를 추상화 한 적이 있다.  
Traversable functor가 Foldable의 접기자료구조 처럼 연산처리가 비슷하다.  
이는 Traverable이 Foldable을 구현 할 수 있음을 암시한다.  
근대 Foldalbe은 monoid를 받아 monoid의 이항연산의 결과를 사용하기 때문에 monoid가 필요한 반면 Traversable은 Applicative Functor가 필요하다.  
따라서 Monoid 를 받아 Monoid의 이항연산을 이용한 Applicative functor 를 만들어 주는 것이  있으면 Traversable Functor 가 Folable 이게 할 수 있다.  
{% highlight scala %}
trait Traverse[F[_]] extends Functor[F] with Foldable {
  def traverse[G[_]: Applicative, A, B](xs: F[A])(f: A => G[B]): G[F[B]]
  def sequence[G[_]: Applicative, A](xs: F[G[A]]): G[F[A]] =
    traverse(xs)(a => a)

  type Id[A] = A
  val idMonad = new Monad[Id] {
    def unit[A](a: => A) = a
    override def flatMap[A, B](ma: Id[A])(f: A => Id[B]): Id[B] = f(ma)
  }

  def map[A, B](ma: F[A])(f: A => B): F[B] =
    traverse[Id, A, B](ma)(f)(idMonad)
    
    
  type Const[M,B] = M // 자신이 감싸는 형식인수를 무조건 M으로만 돌려 준다.
  implicit def monoidApplicative[M](mo: Monoid[M]) = {
    new Applicative[({type f[x] = Const[M,x]})#f] {
      def unit[A](a: => A): M = mo.zero
      def map2[A,B,C](m1: M, m2: M)(f: (A,B) => C): M = mo.op(m1,m2)
    }
  }
  
  override def foldMap[A, M](xs: F[A])(f: A => M)(mb: Monoid[M]): M =
    traverse[({ type f[x] = Const[M, x] })#f, A, Nothing](xs)(a => f(a))(monoidApplicative(mb))
}

{% endhighlight %}

# Traversable Functor의 융합
lazy 로 인한 자료구조의 한번순회로 여러연산을 할 수 있었고, Monoid에서는 Monoid 합성으로 한번의 순회로 여러연산의 결과를 얻을 수 있었다.  
Traversable Functor또한 한번의 순회로 Application Functor을 중첩하여 적용할 수 있다.  
Application Functor의 중첩 함수를 추가 만들면 된다.
{% highlight scala %}
trait Applicative extends Functor {
  ...
  
  def compose[G[_]](G: Applicative[G]): Applicative[({type f[x] = (F[x],G[x])})#f] = {
    val self = this
    new Applicative[({ type f[x] = (F[x],G[x])})#f] {
      def unit[A](a: => A): (F[A],G[A]) = (self.unit(a),G.unit(a))
      override def apply[A,B](fab: (F[A => B],G[A => B]))(p: (F[A],G[A])) = 
        (self.apply(fab._1)(p._1),G.apply(fab._2)(p._2) )
    }
  }
}

//traverse trait 에서 
def fuse[G[_], H[_], A, B](fa: F[A])(f: A => G[B], g: A => H[B])
  (implicit G: Applicative[G], H: Applicative[H]): (G[F[B]], H[F[B]]) = 
  traverse[({type f[x] = (G[x],H[x])})#f,A,B](fa)(a => (f(a),g(a)))(G compose H)
{% endhighlight %}
위의 코드는 F[A]를 한번만 순회하여 함수 f, g를 적용한 결과를 Applicative 의 합성으로 적용할 수 있다.  

위의 코드에서 G 와 H가 Traversable Functor라면 다음과 같이 Traversable Functor자체를 중첩 시킬 수 있다.  
{% highlight scala %}
def compose[G[_]](implicit G: Traverse[G]): Traverse[({type f[x] = F[G[x]]})#f] = 
  new Traverse[({type f[x] = F[G[x]]})#f] {
    def traverse[M[_]: Applicative, A, B](fa: F[G[A]])(f: A => M[B]) = 
      self.traverse(fa)((ga: G[A]) => G.traverse(ga)(f))
}
{% endhighlight %}

# Monad transformer (모나드 조합기)
## Monad는 Monad composite 되지 않는다.
Applicative Functor는 합성이 된다. 즉 G[F[_]] 처럼 말이다.  
아래 Applicative Functor의 합성코들 보면 다음과 같다.
{% highlight scala %}
def map2[F: Applicative[F], G: Applicative[G],A,B,C](fga: F[G[A]], fgb: F[G[B]])(f: (A,B) => C): F[G[c]] =
  F.map2(fga, fgb)(G.map2(_,_)(f))
{% endhighlight %}
아지만 monad를 보면 그렇지 못하다. 아래는 Monad code이다.
{% highlight scala %}
def flatMap[F: Monad[F],G: Monad[G],A,B](fga: F[G[A]])(f: A => F[G[B]]): F[G[B]] = 
  F.flatMap(fga)(ga => G.flatMap(ga)(a => f(a):??? ))
  //f(a)를 하면 F[G[B]] 가 된다. 이는 type dismatch 가 된다.
{% endhighlight %}
위의 flatMap을 보면 f(a)를 하면 F[G[B]] 가 된다. 이는 G monad의 type과 맞지 않다.  
G[F[B]]가 되야 G.flatMap compile이 된다.  
또한 G[F[B]] 가 되었다 하더라도 F monad의 flatMap의 type과 dismatch가 된다.  
F.flatMap은 A => F[G[B]] 가 되어야 하기 때문이다.  

## Traversable Functor로 Monad 합성하기
traversal functor의 traverse 과 Monad의 join를 생각해보자.  
먼저 traverse
{% highlight scala %}
def traverse[G[_]: Applicative, A, B](xs: F[A])(f: A => G[B]): G[F[B]]
{% endhighlight %}
input -> ouput 을 보면 F[A] + G[B] -> G[F[B]] 로 변환한다.  
이것은 위의 flatMap 에서 a => f(a) 부분을 F[G[B]] 이것을 G[F[B]] 로 변환 할 수 있는 방법이 있다는 것이다.  
G의 유형에 Traversable Functor가 있다고 하고 G.flatMap(ga)(a => f(a)) 을 다음과 같이 바꾸어 보자.  
{% highlight scala %}
//G.flatMap(ga)(a => f(a)) 을 다음과 같이 
F.map(G.traverse(ga)(f))(???)

//G.traverse(ga)(f) : F[G[G[B]]]
//F[G[G[B]]] G의 중첩을 제거하는 것은 Monad의 기본 조합기 set중 join 이있다.
F.map(G.traverse(ga)(f))(ggb => G.join(ggb)) // F[G[B]]
{% endhighlight %}
위의 코드를 보면 G.traverse(ga)(f)의 결과는 F[G[G[B]]] 이다. 이는 G가 중첩이므로 중첩을 제거해야 한다.  
근데 Monad의 기본 조합기 set 중에 join이 있으며 이는 중첩을 제거한다. 따라서 이를 이용하면 되겠다. 그럼 F[G[G[B]]] 은 F[G[B]] 가되고 최종 소스를 보면 다음과 같이 된다.
{% highlight scala %}
F.flatMap(fga)(ga => F.map(G.traverse(ga)(f))(ggb => G.join(ggb)))
{% endhighlight %}

위의 Monad의 composite 조합기를 다음과 같이 만들 수 있다.
{% highlight scala %}
def composeM2[F[_],G[_]](implicit F: Monad[F], G: Monad[G], T: Traverse[G]): Monad[({type f[x] = F[G[x]]})#f] = new Monad[({type f[x] = F[G[x]]})#f] {
  def unit[A](a: => A): F[G[A]] = F.unit(G.unit(a))
  override def flatMap[A,B](fga: F[G[A]])(f: A => F[G[B]]): F[G[B]] = 
    F.flatMap(fga)(ga => F.map(T.traverse(ga)(f))(gga => G.join(gga)))
    //T.traverse(ga)(A => F[G[B]]) -> F[G[G[B]]]
    //G.map(F[G[G[B]]]])(G.join)   -> F[G[B]] 
    //reuslt                       -> F.flatMap(F[G[A]])(ga => F[G[B]]) 
}
{% endhighlight %}
위의 Monad 합성은 G에 대한 Traversable Functor가 있을때 가능 하다.  

Monad의 합성을 공통화 하여 추상으로 만들기 어렵다.  
따라서 Moand마다 합성을 위해 특별하게 작성된 버전을 이용해서 해결하기도 한다. 이를 Moand Transfomer라 고 한다.  
위의 Traversable를 이용한 것도 하나의 Monad transformer 이다.  

# Applicative Functor, Traversable Functor
Applicative Functor는 표현력은 Monad보다 덜 하지만 합성능력이 좋다.  
unit, map을 이용하면 값과 함수들을 승급시킬 수 있으며, map2와 apply를 이용하면 함수를 인수가 더 많은 함수로 승급시킬 수 있다.  
Traversable functor는 간단한 요소들만으로 복잡하게 중첩된 병렬 순회를 할 수 있다.


[^1]: This is a footnote.

[kramdown]: https://kramdown.gettalong.org/
[Simple Texture]: https://github.com/yizeng/jekyll-theme-simple-texture
