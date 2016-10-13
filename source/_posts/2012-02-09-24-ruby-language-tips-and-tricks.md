---
layout: post
title: Ruby tips and tricks
date: 2012-02-09 01:35:44.000000000 -08:00
categories:
- Coding
tags:
- ruby
status: publish
type: post
published: true
---

### Generate random numbers within a given range

    irb(main):019:0> rand(10..20)
    => 12
    irb(main):020:0> rand(10...20) # works with exclusive range
    => 16

### Dump your object using awesome_print

    # Install the gem first
    gem install awesome_print

    irb(main):001:0> require 'ap'
    => true
    irb(main):002:0> ap :a => 1, :b => 'greg', :c => [1,2,3]
    {
        :a => 1,
        :b => "greg",
        :c => [
            [0] 1,
            [1] 2,
            [2] 3
        ]
    }
    => {:a=>1, :b=>"greg", :c=>[1, 2, 3]}

### Concatenating strings

    irb(main):005:0> "abc" + "def"
    => "abcdef"
    irb(main):006:0> "abc".concat("def")
    => "abcdef"
    irb(main):007:0> x = "abc" "def"
    => "abcdef"

### Include modules in a single line

    class MyClass
      include Module1, Module2, Module3
      # However, the modules are included in reverse order. Confusing eh!
    end

### Instance variable interpolation

    irb(main):008:0> @name = "greg"
    => "greg"
    irb(main):009:0> "my name is #{@name}"
    => "my name is greg"
    irb(main):010:0> "my name is #@name"
    => "my name is greg"

I still prefer the curly braces.

### Syntax checking

    $ ruby -c facu.rb 
    facu.rb:12: syntax error, unexpected keyword_end, expecting $end

### Zipping arrays

    irb(main):027:0> names = %w(fred jess john)
    => ["fred", "jess", "john"]
    irb(main):028:0> ages = [38, 47,91]
    => [38, 47, 91]
    irb(main):029:0> locations = %w(spain france usa)
    => ["spain", "france", "usa"]
    irb(main):030:0> names.zip(ages)
    => [["fred", 38], ["jess", 47], ["john", 91]]
    irb(main):031:0> names.zip(ages, locations)
    => [["fred", 38, "spain"], ["jess", 47, "france"], ["john", 91, "usa"]]

### Range into arrays

    irb(main):034:0> (10..20).to_a  # what I used to do
    => [10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20]
    irb(main):035:0> [*10..20]
    => [10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20]

### Using parameter as default value

    irb(main):047:0> def method(a, b=a); "#{a} #{b}"; end
    => nil
    irb(main):048:0> method 1
    => "1 1"
    irb(main):049:0> method 1, 2
    => "1 2"

### Put regex match in a variable

    irb(main):058:0> s = "Greg Moreno"
    => "Greg Moreno"
    irb(main):059:0> /(?<first>\w+) (?<second>\w+)/ =~ s
    => 0
    irb(main):060:0> first
    => "Greg"
    irb(main):061:0> second
    => "Moreno"
