---
layout: post
title: "You can't handle the truth"
date: 2016-02-15 16:21:58 -0800
comments: true
categories: 
---

Quick question. What’s the output of this code in Ruby?

    amount = 0
    if amount
      puts 'hey'
    else
      puts ‘nah'
    end

If you answered `nah`, you’re wrong. But it’s fine because this is one of the biggest gotchas for developers who are new to Ruby. Heck, even seasoned developers like myself sometimes forget this. I blame my college CS professors for putting too much `C` syntax in my brain.

Ruby has a simple rule for dealing with boolean values:  everything is true except `false` and `nil`.  This also means that every expression and object in Ruby can be evaluated against true or false. For example, you can have a method `find` that returns an object when it finds one or `nil` otherwise.

    if  o = Customer.find_by(email: ‘stevej@rip.com’)
      puts o.name
    else
      puts ‘not found it'
    end

But it’s a different story when returning a numeric value because 0 evaluates to true.

`false` and `nil` can also be a common source of confusion because you have 2 values that can be false.  Consider the default behaviour of  Hash, which returns nil if the key does not exist. If you only factor in the `nil` scenario, you will have a problem when a key returns a `false` value - a common scenario with code that handles configuration or settings.
In the case below, this will output `missing key`

    h = {'a' => 1, 'b' => false}
    key = ‘b'
    if h[key]
      puts 'found a value'
    else
      puts 'missing key'
    end

If that’s enough confusion for you, consider this:  `true`, `false`, and `nil` are just instances of a class.

    irb> true.class
    => TrueClass
    irb> false.class
    => FalseClass
    irb> nil.class
    => NilClass

They are global variables but you can’t set any value to it which is fine. Otherwise, there will be chaos!

    irb> true = 1
    SyntaxError: (irb):18: Can't assign to true
    true = 1

But, this is Ruby and we can always introduce chaos. Matz, the creator of Ruby,  has given us this much power because he trusts that we know what we are doing.

    irb> class Bad
    irb>   def ==(other)
    irb>     true
    irb>   end
    irb> end

    irb> false == Bad.new
    => false
    irb> Bad.new == false
    => true

What the heck just happened?  Well, `==` is just another method call - the first is for the `FalseClass` instance while the second is for the `Bad` instance.

If you have been using Ruby for a while and wants to become better at it, I suggest you
get a copy of `Effective Ruby` by Peter Jones.
