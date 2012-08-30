---
layout: post
title: Javascript and the unwanted octal number
---

h1. {{ page.title }}

p(meta). 30 Aug 2012

This one cost me a "CoderByte":http://coderbyte.com challenge today.
The challenge was, at a glance, simple enough: make a function that takes a binary number as input return the corresponding decimal value.
That's easy in Javascript: just use @parseInt@ and pass it the number and the base.
The problem was that some of the test cases started with *0*: for instance, *011* (decimal *3*).
To Javascript, that's a base-8 number (octal), so the @parseInt@ would return *9* (base 10) instead of *3*.
I tried a couple of things (like casting it to string before calling @parseInt@ or surrounding it with single quotes) before searching for help, which I found in CoderByte's own forum.
There, another user had asked for help with exactly the same problem to which user _64kps_ answered with this:

{% highlight js %}
return parseInt(num.toString(("" + num).match("[^01]") ? 8 : 10), 2);
{% endhighlight %}

As you can see, he also casts the number to string (@"" + num@) but next he checks if the resulting value starts with a zero or with an one (@.match("^[01]")@). If it starts with a zero, he then uses 8 as the base; otherwise, he uses 10 as the base. This base is then passed into the @toString@ method and the resulting value is finally ready to be passed to @parseInt@. In more steps, this would be:

{% highlight js %}
var base = ("" + num).match("[^01]") ? 8 : 10;
var valueToConvert = num.toString(base);
var result = parseInt(valueToConvert);
return result;
{% endhighlight %}

You can read the original forum thread "here":http://forums.coderbyte.com/viewtopic.php?f=6&t=70&p=186#p145.
