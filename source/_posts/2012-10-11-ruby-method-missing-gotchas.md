---
layout:    post
title:     "#ruby method_missing gotchas"
comments:  true
---

### Forgetting 'super' with 'method_missing'

'method_missing' is one of Ruby's power that makes frameworks like Rails seem magical. When you call a method in an object (or "send a message to the object"), the object executes the first method it finds. If the object can't find the method, it complains. This is pretty much what every modern programming language does. Except in Ruby you can guard against a non-existent method call by having the method 'method_missing' in your object. If you are using Rails, this technique enables dynamic record finders like User.find_by_first_name.

    require "rspec"

    class RadioActive
      def to_format(format)
        format
      end

      def method_missing(name, *args)
        if name.to_s =~ /^to_(\w+)$/
          to_format($1)
        end
      end
    end

    describe RadioActive do
      it "should respond to to_format" do
        format = stub
        subject.to_format(format).should == format
      end

      it "should respond to to_other_format" do
        subject.to_other_format.should == "other_format"
      end

      it "should raise a method missing" do
        expect do
          subject.undefined_method
        end.to raise_error
      end
    end


However, improper use of 'method_missing' can introduce bugs in your code that would be hard to track. To illustrate, our example code above intercepts methods whose name are in the 'to_name' format. It works fine as our tests tell us except when we try to call an undefined method that does not follow the "to_name" format. The default behavior for undefined method is for the object to raise a NoMethodError exception.

    $ rspec method_missing_gotcha-01.rb
    ..F

    Failures:

      1) RadioActive should raise a method missing
         Failure/Error: expect do
           expected Exception but nothing was raised
         # ./method_missing_gotcha-01.rb:30:in `block (2 levels) in <top (required)>'

    Finished in 0.00448 seconds
    3 examples, 1 failure

    Failed examples:

    rspec ./method_missing_gotcha-01.rb:29 # RadioActive should raise a method missing

You can easily catch this bug if you have a test. It would be a different story if you just use your class straight away.

    irb(main):001:0> require './method_missing_gotcha-01.rb'
    => true
    irb(main):002:0> r = RadioActive.new
    => #<RadioActive:0x007fd232a4d8a8>
    irb(main):003:0> r.to_format('json')
    => "json"
    irb(main):004:0> r.to_json
    => "json"
    irb(main):005:0> r.undefined
    => nil

The undefined method just returns nil instead of raising an exception. When we defined our method_missing, we removed the default behavior accidentally. Oops!

Fortunately, the fix is easy. There is no need to raise the 'NoMethodError' in your code. Instead, simply call 'super' if you are not handling the method. Whether you have your own class or inheriting from another, do not forget to call 'super' with your 'method_missing'. And that would make our tests happy :)

    --- 1/method_missing_gotcha-01.rb
    +++ 2/method_missing_gotcha-02.rb
    @@ -9,6 +9,8 @@ class RadioActive
       def method_missing(name, *args)
         if name.to_s =~ /^to_(\w+)$/
           to_format($1)
    +    else
    +      super
         end
       end

    $ rspec method_missing_gotcha-02.rb
    ...

    Finished in 0.00414 seconds
    3 examples, 0 failures

Calling 'super' is not just for 'missing_method'. You also need to do the same for the other hook methods like 'const_missing', 'append_features', or 'method_added'.

### Forgetting respond_to?

When we modified 'method_missing', we are essentially introducing ghost methods. They exist but you cannot see them. You can call them spirit methods if that suits your beliefs. In our example, we were able to use a method named 'to_json' but if we look at the list of methods defined for RadioActive, we will not see a 'to_json'.

    irb(main):002:0> RadioActive.instance_methods(false)
    => [:to_format, :method_missing]
    irb(main):003:0> r = RadioActive.new
    => #<RadioActive:0x007f88b2a151c0>
    irb(main):004:0> r.respond_to?(:to_format)
    => true
    irb(main):005:0> r.respond_to?(:to_json)
    => false

Before we introduce a fix, let us first write a test that shows this bug. It's TDD time baby!

    @@ -32,4 +34,8 @@ describe RadioActive do
         end.to raise_error
       end

    +  it "should respond_to? to_other format" do
    +    subject.respond_to?(:to_other_format).should == true
    +  end
    +
     end

    ...F

    Failures:

      1) RadioActive should respond_to? to_other format
         Failure/Error: subject.respond_to?(:to_other_format).should == true
           expected: true
                got: false (using ==)
         # ./method_missing_gotcha-02.rb:38:in `block (2 levels) in <top (required)>'

    Finished in 0.00444 seconds
    4 examples, 1 failure

    Failed examples:

    rspec ./method_missing_gotcha-02.rb:37 # RadioActive should respond_to? to_other format

The fix is every time you modify 'method_missing', you also need to update 'respond_to?'. And don't forget to include 'super'.

    +  def respond_to?(name)
    +    !!(name.to_s =~ /^to_/ || super)
    +  end
    +
     end

And with that, we are all green.

    ....

    Finished in 0.00443 seconds
    4 examples, 0 failures
