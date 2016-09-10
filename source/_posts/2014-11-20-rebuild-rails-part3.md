---
layout: post
title: 'Rebuild Rails Part 3'
date: 2014-11-20
categories:
- Coding
tags: []
status: publish
type: post
published: true
---

Let's continue our rebuild rails series by supporting controllers. We simplify things
by assuming a `controller/action` path format and only support `GET` request.

We adjust our `.call` implementation to handle the new path format. We also introduce
another method `#render_controller_action` that inspects the `path_info`
and instantiates the right controller using Rails `NameController` convention.


    # lib/tracks.rb
    def self.call(env)
      path_info = env["PATH_INFO"]
      if  path_info == "/"
        text = Tracks::Controller.render_default_root
      else
        text = Tracks::Controller.render_controller_action(env)
      end

      [200, {"Content-Type" => "text/html"}, [text] ]
    end

    # lib/tracks/controller.rb
    def self.render_controller_action(env)
      path_info = env["PATH_INFO"]
      _, controller, action, after = path_info.split("/")

      controller_class_name = controller.capitalize + "Controller"
      controller_class = Object.const_get(controller_class_name)
      controller_class.new.send(action)
    end

    $ ruby spec/application_spec.rb
    Run options: --seed 55835
    # Running:

    .E

    Finished in 0.015837s, 126.2865 runs/s, 126.2865 assertions/s.

      1) Error:
    CrazyApp::Application#test_0002_should respond with different path:
    NameError: uninitialized constant PostsController
        /code/crazy/lib/tracks/controller.rb:13:in `const_get'

When we run the test, you see it failed on `PostsController` which is what
we expect since we haven't implemented `PostsController` yet. Let's add the
controller now.

  # app/controllers/posts_controller.rb
  class PostsController < Tracks::Controller
    def index
      "hello from tracks /posts/index"
    end
  end

  # config/application.rb
  require "./app/controllers/posts_controller"

  $ ruby spec/application_spec.rb
  Run options: --seed 2475

  # Running:
  ..

  Finished in 0.019152s, 104.4277 runs/s, 208.8555 assertions/s.

  2 runs, 4 assertions, 0 failures, 0 errors, 0 skips

# Automatic loading

Our tests pass but something's not right. In Rails, there is no need to require
every controller (or pretty much anything) to make it work. To support this feature in
our framework, we need 2 things:

* convert PostsController to posts_controller.rb; and
* auto-require 'posts_controller'

To tie these 2 together, we tap into `Object.const_missing` so our framework would
know if a class has been used but not yet loaded.

We also update the `$LOAD_PATH` to include `app/controllers` folder so Ruby knows
where to look.

    # lib/tracks.rb
    require File.expand_path("../tracks/helper", __FILE__)
    require File.expand_path("../tracks/object", __FILE__)

    # config/application.rb
    require './lib/tracks'
    $LOAD_PATH << File.expand_path("../../app/controllers", __FILE__)

    module CrazyApp
      class Application < Tracks::Application
      end
    end

    # lib/tracks/helper.rb
    module Tracks
      module Helper
        def self.to_underscore(string)
          string.scan(/[A-Z][a-z]+/).
          join('_').
          downcase
        end
      end
    end

    # lib/tracks/object.rb
    class Object
      def self.const_missing(c)
        require Tracks::Helper.to_underscore(c.to_s)
        const_get(c)
      end
    end

Our implementation of `.to_underscore` is limited compared to what's supported in Rails.

    irb(main):019:0> s = "PostsController"
    => "PostsController"
    irb(main):020:0> s.scan(/[A-Z][a-z]+/).join('_').downcase
    => "posts_controller"

Also, since we call `const_get` inside `const_missing`, you will run into serious
trouble if your file does not contain the expected class. Try changing the name of the
class inside `app/controllers/posts_controller.rb` into something else and you will get this
error.

    $ ruby spec/application_spec.rb
    Run options: --seed 60433

    # Running:

    .E
    Finished in 0.043152s, 46.3478 runs/s, 46.3478 assertions/s.

      1) Error:
    CrazyApp::Application#test_0002_should respond with different path:
    SystemStackError: stack level too deep
        /Users/greg/.rbenv/versions/2.0.0-p247/lib/ruby/2.0.0/forwardable.rb:174

    2 runs, 2 assertions, 0 failures, 1 errors, 0 skips
