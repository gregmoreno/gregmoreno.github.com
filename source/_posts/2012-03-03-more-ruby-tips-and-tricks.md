---
layout: post
title: More Ruby tips and tricks
date: 2012-03-03 01:50:39.000000000 -08:00
categories:
- Coding
tags:
- ruby
status: publish
type: post
published: true
---

### String to number conversion gotcha

    >> Float('3.14159')
    => 3.14159 
    >> '3.14159'.to_f
    => 3.14159 

    # However, Float() method will return an exception if given
    # a bad input while to_f() will ignore everything from the 
    # offending character.

    >> Float('3.x14159')
    ArgumentError: invalid value for Float(): "3.x14159"
      from (irb):4:in 'Float'
      from (irb):4

    >> '3.x14159'.to_f
    => 3.0


    # Similar case with to_i() and Integer().

    >> Integer('19x69')
    ArgumentError: invalid value for Integer(): "19x69"
      from (irb):15:in 'Integer'
      from (irb):15
      from /Users/greg/.rvm/rubies/ruby-1.9.2-p0/bin/irb:17:in '<main>'

    >> '19x69'.to_i
    => 19


### Case insensitive regular expression

    # Regex is case sensitive by default.
    # Adding 'i' for insensitive match
    puts 'matches' if  /AM/i =~ 'am'

### Hash is ordered in 1.9

    # new syntax in 1.9
    h = {first: 'a', second: 'b', third: 'c'}

    # hashes in 1.9 are ordered
    h.each do |e|
      pp e
    end

### Filter a list using several conditions

    conditions = [
        proc { |i| i > 5 },
        proc { |i| (i % 2).zero? },
        proc { |i| (i % 3).zero? }
      ]

    matches = (1..100).select do |i|
      conditions.all? { |c| c[i] }
    end

### Randomly pick an element from an array

    >> [1,2,3,4,5].sample
    => 2 
    >> [1,2,3,4,5].sample
    => 1 

    # pick 2 random elements
    >> [1,2,3,4,5].sample(2)
    => [1, 5]

### List methods unique to a class

    # List all instance methods that starts with `re` including those inherited by String.

    >> String.instance_methods.grep /^re/
    => [:replace, :reverse, :reverse!, :respond_to?, :respond_to_missing?] 

    # List methods unique to String, i.e. not include
    # those defined by its ancestors.

    >> String.instance_methods(false).grep /^re/
    => [:replace, :reverse, :reverse!]

### Globbing key-value pairs

    >> h = Hash['a', 1, 'b', 2]
    => {"a"=>1, "b"=>2}

    >> h = Hash[ [ ['a', 1], ['b', 2] ] ] 
    => {"a"=>1, "b"=>2}

    >> h = Hash[ 'a' => 1, 'b' => 2 ]
    => {"a"=>1, "b"=>2}

    # The first form is very useful for globbing key-value pairs in Rails’ routes. For example, if you have the following:

    # route definition in Rails 3
    match 'items/*specs' => 'items#specs'

    # sample url
    http://localhost:3000/items/year/1969/month/7/day/21

    # params[:specs] will be set

    >> params[:specs]
    => "year/1969/month/7/day/21"

    >> h = Hash[*params[:specs].split('/')]
    => {"year"=>"1969", "month"=>"7", "day"=>"21"}
