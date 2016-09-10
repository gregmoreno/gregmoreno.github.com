---
layout: post
title: '#ruby Counting Vowels'
date: 2014-11-13
categories:
- Coding
tags: []
status: publish
type: post
published: true
---

Just saw a simple exercise in my Facebook feed and I thought I give it a shot. The problem is simple:

> Write a function that returns the number of vowels in the string.

Here's my ruby solution:

      require 'minitest/autorun'

      def vowel_count(s)
        vowels = %w[a e i o u]
        s.to_s.scan(/\w/).select { |i| i if vowels.include?(i.downcase) }.count
      end

      describe "#vowel_count" do
        it "should count upcase lowercase" do
          test = "I wanted to be an astronaut"
          vowel_count(test).must_equal 10
        end

        it "should be zero for empty string" do
          vowel_count("").must_equal 0
        end

        it "should be zero for nil" do
          vowel_count(nil).must_equal 0
        end
      end

Sounds simple, right? But there are subtle things you should watch out for.

* *Upper and lower cases* may seem trivial but programmers are often bitten by these when comparing strings.
* An initial solution would be to access each character via [index] and increment a counter for vowels.
Here is where familiarity with your language's libraries becomes useful.
While I didn't get the right method initially, I know Ruby's String library offers a way
to extract regex matches. From then on, it's just a matter of using `Enumerable#select`
which is a common Ruby idiom for filtering elements.
* Having tests even for a simple code is a good discipline to have. My initial test only
covers the functional requirement. When I added the case of nil it quickly showed
the flaw in my code, which brings me to my next point.
* Produce sensible results as much as possible. While you can argue the requirement
states a string and not a nil, it is good habit to defend your code in case the caller
passed an invalid value. Hence, I converted the parameter to a string to ensure the
rest of the code is working with a string object and it gives a sensible result even
if the passed parameter is not a string.

### Minimalist testing

If you are working with Rails' for a while, you probably been pampered with Rails seamless
integration with testing frameworks you'll be forgiven if you think these support
are only available within Rails.

Ruby comes with `minitest/autorun` that supports a minimalist testing framework. Just require in your code and you are
good to go with rspec-style testing right off the bat.

    $ ruby vowelcount.rb
    Run options: --seed 47907

    # Running:

    ...

    Finished in 0.001155s, 2597.4026 runs/s, 2597.4026 assertions/s.

    3 runs, 3 assertions, 0 failures, 0 errors, 0 skips
