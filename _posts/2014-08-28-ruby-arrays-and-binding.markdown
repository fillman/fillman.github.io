---
layout: post
title:  "Ruby arrays and binding"
date:   2014-08-28 19:40:28
---

While I was reading rails code, I figured out interesting thing about ruby arrays and how they bind values to variables while iterating.

You know the basic thing:

{% highlight ruby %}
[1, 2, 3].each { |i| puts i } # print 1 \n 2 \n 3
{% endhighlight %}

Thats obvious, I know, but what I did not knew is:

{% highlight ruby %}
[[1, 2], [3,4], [5,6]].each do |first, last|
  puts first # => 1
  puts last  # => 2
end
{% endhighlight %}

It dynamically binds array values to arity variables passed into blocks.

As you guessed:

{% highlight ruby %}
[[1, 2], [3,4], [5,6]].each do |first, last, none|
  ...
  puts none # => nil
end
{% endhighlight %}

This is really neat stuff, rails uses it in activesupports +lazy_load_hooks+ module:

{% highlight ruby %}
def self.on_load(name, options = {}, &block)
  @loaded[name].each do |base|
    execute_hook(base, options, block)
  end
 
  # @load_hooks is a simple hash, that returns array passing any key,
  # and we insert array into that key, containing block and options,
  # when we'll call +run_load_hooks+ it will iterate over those arrays binding
  # block to hook variable and options to options variable, watch next method.
  @load_hooks[name] << [block, options]
end
 
...
 
def self.run_load_hooks(name, base = Object)
  @loaded[name] << base
  # Look closely to @load_hooks, the key :name will return array:
  # [ [block, options], [block, options], ... ]
  # so hook variable gets the block and options speak for itself
  @load_hooks[name].each do |hook, options|
    execute_hook(base, options, hook)
  end
end

{% endhighlight %}