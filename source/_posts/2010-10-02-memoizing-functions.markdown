---
layout: post
title: Memoizing Functions
tags:
- Functional Programming
- Ruby
status: publish
type: post
published: true
meta:
  jabber_published: '1286050631'
  _edit_last: '16929416'
  _wp_old_slug: ''
  email_notification: '1286050632'
---

I've been going  through some parts of [The Ruby Programming Language](http://oreilly.com/catalog/9780596516178/) book recently, the best book about Ruby language I've read and here are some interesting bits about memoization that I found:

> Memoization is a functional programming term for caching the results of
> a function invocation. If a function always returns the same value when
> passed the same arguments, if there is reason to believe that the same
> arguments will be used repeatedly, and if the computation it performs
> is somewhat expensive, then memoization may be a useful optimization...

While I was reading about basic functional programming concepts, I picked up some FP terminology. For instance *"a function always returns the same value when passed the same arguments"* is called **referential transparency** in FP lingo, which is important concept since it assumes variables are non mutable. With this feature, chances of a bug occurring as a result of so called (mutability) **side effects** cease to exist.

The book offers Ruby solution for the memoization of the Proc and Method objects:

{% highlight ruby %}
module Functional
  def memoize
    cache = {}
    lambda { |*args|
      unless cache.has_key?(args)
        cache[args] = self[*args]
      end
      cache[args]
    }
  end
end
module Proc; include Functional; end
module Method; include Functional; end
{% endhighlight %}

I've removed comments for the sake of readability. Here is basically what function does: if there is no results for the particular <code>args</code>
a procedure/method will be executed and the results cached and returned, otherwise already cached results will be returned.
What I find unusual here is that the entire array of arguments is the hash key.

I thought it would be interesting to excercise memoizing technique and  decided to try it out on Fibonnaci sequences:

{% highlight text %}
0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, ...
{% endhighlight %}

Generally, this sequence can be defined in the form of the function F(n) such that:

{% highlight text %}
F(n) = 0, if n = 0
F(n) = 1, if n = 1
F(n) = F(n-1) + F(n+2), otherwise.
{% endhighlight %}

In Ruby this definition can be translated in straightforward, but,  from the performance point of view,  terrible lambda function.

{% highlight ruby %}
# Fibonnaci numbers computed with
# a tree recursive algorithm without memoization.
fib_wo_mem = lambda do |n|
  return 0 if n == 0
  return 1 if n == 1
  fib_wo_mem(n - 1) + fib_wo_mem(n - 2)
end
{% endhighlight %}

We can see why this function is so terrible. It will do a lot of redundant computation. Only fib_wo_mem(1) and fib_wo_mem(0) will
be computed F(n+1) times, and if we know that F(n) grows exponentially WRT n, than we get an idea of how
bad this can be. Even with such a bad choise our friend memoization might save us to some extent.

First we need to slightly modify the above fib_wo_mem function:

{% highlight ruby %}
fib_with_mem = lambda do |n|
    return 0 if n == 0
    return 1 if n == 1
    fib_with_mem(n - 1) + fib_with_mem(n - 2)
  end.memoize
{% endhighlight %}

Let's compare these two:

{% highlight ruby %}
Benchmark.bmbm{ |x|
   x.report('fib_with_mem'){ fib_with_mem[20] }
   x.report('fib_wo_mem'){ fib_wo_mem[20] }
}

# Rehearsal -----------------------------------------------
# fib_with_mem   0.000000   0.000000   0.000000 (  0.000175)
# fib_wo_mem     0.400000   0.000000   0.400000 (  0.413050)
# --------------------------------------- total: 0.400000sec
#                    user     system       total        real
# fib_with_mem   0.000000   0.000000   0.000000 (  0.000019)
# fib_wo_mem     0.410000   0.010000   0.420000 (  0.410216)
{% endhighlight %}

Because everything is already cached after the first execution "fib_with_mem" is about 10 times faster the second time.
So it is fair  to  take into account just Rehearsal times.  Even just for the first execution the difference is striking.
Fibonnaci function with memoization is more than 2,000 times faster than its w/o memoization counterpart.
(I used here number 20 because with anything bigger <code>fib_wo_mem</code> function gets unbearably slow. You can try it.)

After this we may be interested to see  how this recursive, not_appropriate_solution,
compares with a more efficient implementation of fibonnaci function.

More efficient solution may be to use pair of integers a and b with iterative, parallel assignments: a = a + b, b = a.
After n such iterations a and b should be equal to F(n+1) and F(n) respectively. In order to come to this solution
we only need to apply basic rule of the sequence F(n) = F(n-1) + F(n-2) in n iterations. Each iteration will take two last numbers
from the sequence as an input, and produce next two as an output which than will serve as an input for next iteration...

Here is how it can be implemented:

{% highlight ruby %}
# linear iteration fibonaci
lin_fib_iter = lambda do |a, b, n|
  return b if n == 0
  lin_fib_iter[a+b, a, n-1]
end
# While lin_fib_iter is a recursive procedure,
# computation itself is iterative.
{% endhighlight %}

Let's crunch some numbers and do benchmarking:

{% highlight ruby %}
Benchmark.bmbm{ |x|
  x.report('fib_with_mem'){ fib_with_mem[1000] }
  x.report('lin_fib_iter'){ lin_fib_iter[1, 0, 1000] }
}

# Rehearsal -------------------------------------------
# fib_with_mem   0.020000   0.000000   0.020000 ( 0.017360)
# lin_fib_iter   0.000000   0.000000   0.000000 ( 0.001376)
# ---------------------------------- total: 0.020000sec
#                    user     system      total        real
# fib_with_mem   0.000000   0.000000   0.000000 ( 0.000041)
# lin_fib        0.000000   0.000000   0.000000 ( 0.001345)
{% endhighlight %}

Even for a small number such as 1000, iterative solution is more than 10 times faster (for the first execution). This is expected, since the iterative solution has linear, while the first solution has exponential growth WRT n. Although the tree-recursive-memoized version is about 30 times faster the second time (everything is already cached) it is not right approach for the given problem (not just because "stack level too deep" error begins to occur already at n = 1500 ). This does not  imply that tree recursive processes are useless. On contrary they can be natural solutions for some classes of problems.

*Note: Implementations of the Fibonnaci functions are translated from their orginal Scheme versions which can be found in
the one of the CS's classics:  [Structure and Interpretation of Computer Programs](http://mitpress.mit.edu/sicp/full-text/book/book.html).*

