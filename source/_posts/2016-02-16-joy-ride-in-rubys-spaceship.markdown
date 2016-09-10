---
layout: post
title: "Joy Ride in Ruby's spaceship"
date: 2016-02-16 20:52:04 -0800
comments: true
categories:
---

Ruby's `<=>` operator is commonly referred to as the spaceship in the community. You may not have
played with it directly but I bet you have relied on it a lot of times. Because, every time you use
sort an array, you are tapping in to the spaceship operator.

Now, why would you care? Because sometimes we need to sort things which does not have a natural ordering
computers are accustomed to. Take for example sorting clothes that use S, M, and L to refer to their sizes.
And to add more fun, how about putting XS, XL, and XXL into the mix.

Let's start with a bare class and see what happens.

    require 'minitest/autorun'

    class Size
      attr_reader :size

      def initialize(size)
        @size = size.to_s.upcase
      end

      def to_s
        @size
      end
    end

    describe Size do
      let(:sizes) { %w[L S M].map { |s| Size.new(s) } }
      it { sizes.sort.map(&:to_s).must_equal %w[S M L] }

      let(:a) { Size.new('S') }
      let(:b) { Size.new('M') }

      it { (a > b).must_equal false }
      it { (a < b).must_equal true }
    end

    $> ruby size.rb                                                                                                     [2.1.2]
    Run options: --seed 37770

    # Running:

    EEE

    Finished in 0.001207s, 2486.1356 runs/s, 0.0000 assertions/s.

      1) Error:
    Size#test_0001_anonymous:
    ArgumentError: comparison of Size with Size failed
        size.rb:24:in `sort'
        size.rb:24:in `block (2 levels) in <main>'


      2) Error:
    Size#test_0002_anonymous:
    NoMethodError: undefined method `>' for #<Size:0x007f8a7bae6c50 @size="S">
        size.rb:29:in `block (2 levels) in <main>'


      3) Error:
    Size#test_0003_anonymous:
    NoMethodError: undefined method `<' for #<Size:0x007f8a7bae6390 @size="S">
        size.rb:30:in `block (2 levels) in <main>'

    3 runs, 0 assertions, 0 failures, 3 errors, 0 skips

Our test failed and that's good news, isn't it? The first failure is because the default
implementation of `<=>` doesn't do all the comparison required to make `sort` work. To make this
work, we need to implement a `<=>` that returns the following:

* nil if the comparison does not makes sense
* -1 if left side is less than right side
* 1 if left side is greater than right side
* 0 if left and right are the same

Let's update our code to use the spaceship.


    class Size
      attr_reader :size

      SIZES = %w[S M L].freeze

      def initialize(size)
        @size = size.to_s.upcase
      end

      def to_s
        @size
      end

      def <=>(other)
        position <=> other.position
      end

      protected

      def position
        SIZES.index(size)
      end
    end


Some things to ponder in our implementation.

* In Ruby, operator calls are just method calls where the left side is the receiver and right side is
the argument. In other words, this is  `a <=> b` is the same as `a.<=>(b)`
* It is common practice to call the argument as `other` as you know the other object :)
* We leverage an existing implementation of `<=>`. The method `#index` returns the position of an element
in the array, which is a `Fixnum`. The `Fixnum` class already knows how to compare numbers.
* We use `protected` to hide an implementation detail but at the same time allow us to use it within instance
methods of objects of the same class.

Now, how about the other test failures? Do we need to implement the `<` and `>` operators as well?
Fortunately, Ruby got our back. We just need to include the module [`Comparable`](http://ruby-doc.org/core-2.3.0/Comparable.html)
and we're good. But wait, there's more! By including the `Comparable` module, we also get 
`<=`, `>=`, and `==` for free.

Here's the full implementation with additional test scenarios, including a reverse sort.


    require 'minitest/autorun'

    class Size
      include Comparable

      SIZES = %w[S M L].freeze

      attr_reader :size

      def initialize(size)
        @size = size.to_s.upcase
      end

      def to_s
        @size
      end

      def <=>(other)
        position <=> other.position
      end

      protected

      def position
        SIZES.index(size)
      end
    end

    describe Size do
      let(:sizes) { %w[L S M].map { |s| Size.new(s) } }

      it { sizes.sort.map(&:to_s).must_equal %w[S M L] }
      it { sizes.sort { |a,b| b <=> a }.map(&:to_s).must_equal %w[L M S] }

      let(:a) { Size.new('S') }
      let(:b) { Size.new('M') }

      it { (a > b).must_equal false }
      it { (a < b).must_equal true }
      it { (a >= b).must_equal false }
      it { (a <= b).must_equal true }
      it { (a == b).must_equal false }
    end

    $> ruby size.rb                                                                                                     [2.1.2]
    Run options: --seed 44807

    # Running:

    .......

    Finished in 0.001490s, 4698.8003 runs/s, 4698.8003 assertions/s.

    7 runs, 7 assertions, 0 failures, 0 errors, 0 skips

*Exercise:*  Software versions often follow the convention `major.minor.patch`. Create a `Version` class
that takes a string version, e.g. "10.10.3" and implement the `<=>` operator.

I'm currently reading `Effective Ruby` by Peter Jones and this post is based on Item 13: Implement
Comparison via "<=>" and the Comparable Module.
