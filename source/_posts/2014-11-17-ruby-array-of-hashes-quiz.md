---
layout: post
title: '#ruby Array of hashes quiz'
date: 2014-11-17
categories:
- Coding
tags: []
status: publish
type: post
published: true
---

Found this interesting ruby quiz from [AlphaSights](http://www.alphasights.com/careers/experienced/positions/new-york/6-ruby-on-rails-developer/apply).
Given an array of hashes, collapse into an array of hashes containing one entry per day.
And you can only reference the `:time` key and not the rest.

    log = [
      {time: 201201, x: 2},
      {time: 201201, y: 7},
      {time: 201201, z: 2},
      {time: 201202, a: 3},
      {time: 201202, b: 4},
      {time: 201202, c: 0}
    ]

    # result should be
    [
      {time: 201201, x: 2, y: 7, z: 2},
      {time: 201202, a: 3, b: 4, c: 0},
    ]

The first thing came to mind is to use [`Enumerable#group_by`](http://www.ruby-doc.org/core-2.1.1/Enumerable.html#method-i-group_by)

    grouped = log.group_by { |i| i[:time] }
    collapsed = grouped.collect do |t, a|
      no_time_h = a.inject({}) do |others, h|
        others.merge h.reject { |k, v| k.to_sym == :time }
      end

      {time: t}.merge(no_time_h)
    end

    puts collapsed.inspect

However, after reading this a couple of times, I still find the solution hard to follow.
For starter, `group_by` returns a hash where the values are an array of hashes which
brings me back to the original problem even though it is already grouped by time.
That I feel made the rest of the code more complicated.

    # result of group_by
    {201201=>[{:time=>201201, :x=>2}, {:time=>201201, :y=>7}, {:time=>201201, :z=>2}], 201202=>[{:time=>201202, :a=>3}, {:time=>201202, :b=>4}, {:time=>201202, :c=>0}]}

For my second version, I simply loop into the array and compose the hash using `:time` as the key.
Afterwards, use the `key-value` pair to compose the resulting array. The code may be longer but
it is more readable. Remember, [Correct, Beautiful, Fast (in That Order)](https://www.safaribooksonline.com/library/view/beautiful-code/9780596510046/ch05.html).

    hash_by_time = {}
    log.each do |h|
      time = h[:time]
      others = h.reject { |k,v| k.to_sym == :time }

      if hash_by_time[time]
        hash_by_time[time].merge! others
      else
        hash_by_time[time] = others
      end
    end

    collapsed = hash_by_time.collect do |k, v|
      {time: k}.merge(v)
    end
