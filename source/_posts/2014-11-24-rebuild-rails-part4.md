---
layout: post
title: 'Rebuild Rails Part 4'
date: 2014-11-24
categories:
- Coding
tags: []
status: publish
type: post
published: true
---

Now, it is time to build real pages in our super duper Tracks framework. We will support ERB and to do that
we need the [erubis](https://rubygems.org/gems/erubis) gem.

    # app/controllers/posts_controller.rb
    class PostsController < Tracks::Controller
      def index
        locals = { title: "/posts/index" }
        render template: "posts/index", locals: locals
      end
    end

    # app/views/posts/index.html.erb
    hello from tracks <%= title %>

    # lib/tracks/controller.rb
    def render(options)
      template_name = options.fetch(:template)
      locals = options.fetch(:locals)

      filename = File.join "app/views", "#{template_name}.html.erb"
      template = File.read(filename)
      erb = Erubis::Eruby.new(template)
      erb.result locals
    end

    $ ruby spec/application_spec.rb
    Run options: --seed 12593

    # Running:

    ..

    Finished in 0.020498s, 97.5705 runs/s, 195.1410 assertions/s.

    2 runs, 4 assertions, 0 failures, 0 errors, 0 skips


Our tests show we are good but it's not even close to Rails' "magic". First, let's
use the `controller/action` convention, i.e. if no template is passed to the `render` method,
we should use the `app/views/posts/index.html.erb` template. We also modify our `Controller` to save
the `env` data passed by `Rack` because we need to check the path.

    # lib/tracks/controller.rb
    module Tracks
      class Controller
        attr_reader :env, :controller_name, :action_name

        def initialize(env)
          @env = env
          extract_env_info
        end

        def extract_env_info
          _, @controller_name, @action_name, after = path_info.split("/")
        end

        def path_info
          env["PATH_INFO"]
        end

        def extract_template_name
          "#{controller_name}/#{action_name}"
        end
      end
    end

We then update our `render` method to check for the template if it is not passed.

    # lib/tracks/controller.rb
         def render(options)
    -      template_name = options.fetch(:template)
    +      template_name = options.fetch(:template) { extract_template_name }
           locals = options.fetch(:locals)

           filename = File.join "app/views", "#{template_name}.html.erb"
    @@ -29,6 +43,5 @@ module Tracks
           erb.result locals
         end

Our next modification involves using the @instance_variables to pass values from the
controller to the view files. To do that, we just need to pass the current `binding` to
the `eruby` instance and it should pickup the instance variables we have in the controller.

In Rails, it is a bit more involved. There is a concept of [view context](https://github.com/rails/rails/blob/0c5552a3dd28e35cce64462765cc41c5355db0f1/actionpack/lib/abstract_controller/rendering.rb#L84-L86). Rails
collects the instance variables from the controller, then duplicates the values into the view context.
The Ruby methods `#instance_variables`, `#instance_variable_get`, `#instance_variable_set` allow
Rails to accomplish that.

    def render(options={})
      template_name = options.fetch(:template) { extract_template_name }
      filename = File.join "app/views", "#{template_name}.html.erb"
      template = File.read(filename)
      erb = Erubis::Eruby.new(template)
      erb.result(binding)
    end

We also update our `render` method and controller because we do not need the `locals` parameter.

    # app/controllers/posts_controller.rb
    class PostsController < Tracks::Controller
      def index
        @title = "/posts/index"
        render
      end
    end

    # app/views/posts/index.html.erb
    hello from tracks <%= @title %>

We still have the extra `render` call in our controller. To remove it, we keep track of the call
to render and if there's no rendered result yet, we call `render`.

    diff --git i/app/controllers/posts_controller.rb w/app/controllers/posts_controller.rb
    index f7883e3..d7aa012 100644
    --- i/app/controllers/posts_controller.rb
    +++ w/app/controllers/posts_controller.rb
    @@ -1,6 +1,5 @@
     class PostsController < Tracks::Controller
       def index
         @title = "/posts/index"
    -    render
       end
     end
    diff --git i/lib/tracks/controller.rb w/lib/tracks/controller.rb
    index 0119c1b..3d5d75a 100644
    --- i/lib/tracks/controller.rb
    +++ w/lib/tracks/controller.rb
    @@ -30,7 +30,12 @@ module Tracks

           controller_class_name = controller.capitalize + "Controller"
           controller_class = Object.const_get(controller_class_name)
    -      controller_class.new(env).send(action)
    +      controller_context = controller_class.new(env)
    +      controller_context.send(action)
    +
    +      if controller_context.rendered_string.nil?
    +        controller_context.render
    +      end
    +
    +      controller_context.rendered_string
         end

         def render(options={})
    @@ -38,7 +43,7 @@ module Tracks
           filename = File.join "app/views", "#{template_name}.html.erb"
           template = File.read(filename)
           erb = Erubis::Eruby.new(template)
    -      erb.result(binding)
    +      @rendered_string = erb.result(binding)
         end

We also update our controller and tests to cover the case of using `render` explicitly.

    # app/controllers/posts_controller.rb
    class PostsController < Tracks::Controller
      def index
        @title = "/posts/index"
      end

      def show
        @title = "/posts/index"
        render template: "posts/index"
      end
    end

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
        get "/posts/index"
        last_response.ok?.must_equal true
        last_response.body.strip.must_equal "hello from tracks /posts/index"
      end

      it "should respond with different template" do
        get "/posts/show"
        last_response.ok?.must_equal true
        last_response.body.strip.must_equal "hello from tracks /posts/index"
      end
    end
