---
layout: post
title: 'Rebuild Rails Part 2'
date: 2014-11-19
categories:
- Coding
tags: []
status: publish
type: post
published: true
---

Before we continue with out rebuilding series, we should write some tests first :)

{% highlight ruby %}

# spec/spec_helper.rb
ENV["RAILS_ENV"] ||= "test"

require "rack/test"
require "minitest/autorun"

require File.expand_path("../../config/application", __FILE__)


# spec/application_spec.rb
require_relative "spec_helper"

describe CrazyApp::Application do
  include Rack::Test::Methods

  def app
    CrazyApp::Application
  end

  it "should respond with /" do
    get "/"
    last_response.ok?.must_equal true
    last_response.body.strip.must_equal "hello from index.html"
  end

  it "should respond with different path" do
    get "/posts"
    last_response.ok?.must_equal true
    last_response.body.strip.must_equal "hello from tracks /posts"
  end

end

{% endhighlight %}

Just like your typical Rails test setup, we have a common `spec_helper` file. We use `minitest/autorun` which gives us
rspec-style DSL out of the box. For our test, we need `Rack::Test::Methods` to use `get` and other http methods. We also
need an `app` method that returns our Rack application to make the tests work.

{% highlight bash %}
$ ruby spec/application_spec.rb
Run options: --seed 57256

# Running:
..

Finished in 0.015864s, 126.0716 runs/s, 252.1432 assertions/s.

2 runs, 4 assertions, 0 failures, 0 errors, 0 skips
{% endhighlight %}

We're all good. Awesome!
