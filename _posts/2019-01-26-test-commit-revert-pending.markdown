---
layout: post
title:  "test && commit || revert ; pending"
date:   2019-01-27 02:54:00 +0100
author: Oddmund Strømme
---

People have been pointing out how [TCR][tcr] does not support the _red test_ step of TDD. In TDD you begin with writing a test that does not pass. If you make a test that does not pass in TCR, you lose the test you just wrote.

I have seen various suggested solutions to this. Some would have their TCR script never revert test code – only _implementation_ code. This strikes me as complicated.  **And it feels wrong to hold test code to a lower standard than implementation code.**

I have also seen suggestions where the TCR flow would have different states, where you would not revert if you are in a _red state_. This sounds like regular TDD and only applying TCR to the refactoring step. Which seems pretty reasonable, but it is not what I want to do.

**Instead of changing how I revert, I am changing how I fail.** I want to write a red test that fails, but does not cause any reverting to happen.

I have done this Elixir, but I assume you can do similar things with other languages. I `@tag` my red test with `:pending`.

{% highlight elixir %}
@tag :pending
test "red test" do
  assert 1 == 2
end
{% endhighlight %}

This does not change anything by itself, but allows me to exclude the test from the T in my TCR script:

{% highlight bash %}
mix test --exclude pending
{% endhighlight %}

This makes the failing of the red test a _safe move_, **but it does not protect the test (or any other code) from being reverted by an actual failure**.

I still want the feedback from the failing red test, so after the TCR steps, I run the pending test alone:

{% highlight bash %}
mix test --only pending
{% endhighlight %}

I end up with something like this:

{% highlight bash %}
mix test --exclude pending \
&& git commit -am $1 \
|| git reset --hard \
; mix test --only pending
{% endhighlight %}

Got any thoughts on this? Bark at me on [Twitter](https://twitter.com/jraregris)

[tcr]: https://medium.com/@kentbeck_7670/test-commit-revert-870bbd756864