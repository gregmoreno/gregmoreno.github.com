---
layout: post
title: 'Rebuild Rails Part 1'
date: 2014-11-16
categories:
- Coding
tags: []
status: publish
type: post
published: true
---

I have no plans to build another Rails-clone. Let's leave that work to other smarter people with more time.
But wouldn't it be fun if we can learn how Rails work under the hood and find out what makes it "magical"?
In this post I will only cover what happens when you typed that url until you get an HTML page. We'll simplify further by not using any database
access. If you would like to go deeper and wider, there is a [book devoted entirely to it that I highly recommend](https://rebuilding-rails.com/).

Let's call our application `CrazyApp` and let's build our first web app.

    # config.ru
    app =  proc do |env|
      [200, {'Content-Type' => 'text/html'}, ["hello from crazy app"] ]
    end

    run app

    $ rackup config.ru -p 3000                                                                       [2.0.0-p247]
    Thin web server (v1.6.2 codename Doc Brown)
    Maximum connections set to 1024
    Listening on 0.0.0.0:3000, CTRL+C to stop
    127.0.0.1 - - [16/Nov/2014 20:42:02] "GET / HTTP/1.1" 200 - 0.0008


### It's all about the Rack

Rack is a gem that sits between your framework (e.g. Rails) and Ruby-based
application servers like Thin, Puma, Unicorn, and WEBrick. When you type a url,
it goes through several layers of software until it hits our application which
in this case just returns a `hellow from crazy app`. Rack simplifies the interface
for web servers that we only have to worry about a few things to handle an HTTP request.

* HTTP status, e.g. 200
* Response headers. There are lot of things you can set here but for now let's stick to content-type.
* Actual content. In our case, an HTML page.

Let's look at a boilerplate `config.ru` that you get from Rails.

    # This file is used by Rack-based servers to start the application.

    require ::File.expand_path('../config/environment',  __FILE__)
    run Rails.application

One step in [Rails' bootup process](http://api.rubyonrails.org/classes/Rails/Application.html) is to
define `Rails.application` as `class MyApp::Application < Rails::Application`.
Both `Rails::Application` and `proc` provides a `call` method that is why both `config.ru` works.

Now, let's move our initial `config.ru` code to a different class that we can later extract into
a gem for our framework that we shall call `Tracks`. From here on, we shall follow Rails conventions
and build our gem from there.

    # config.ru
    require './config/application'
    run CrazyApp::Application

    # config/application.rb
    require './lib/tracks'

    module CrazyApp
      class Application < Tracks::Application
      end
    end

    # lib/tracks.rb
    module Tracks
      class Application
        def self.call(env)
          [200, {'Content-Type' => 'text/html'}, ["hello from tracks"] ]
        end
      end
    end


Exit from your rackup process and re-run it because we are not supporting auto-reloading. This time you will see a message from our `Tracks` - the super awesome Rails-like framework.


### Render a default page

We now introduce a very simple root controller and use it to render a default index page. We also modify our route handling
by inspecting the value in `env` object that Rack passed to our framework. The `env` packs a lot of information about
a request and for our routing, we are interested in `PATH_INFO` which is the url after the domain minus the query parameters.

    # lib/tracks/controlle.rb
    module Tracks
      class Controller
        def self.render_default_root
          filename = File.join('public', 'index.html')
          File.read filename
        end
      end
    end


    # lib/tracks.rb
    require File.expand_path('../tracks/controller', __FILE__)

    module Tracks
      class Application
        def self.call(env)
          path_info = env['PATH_INFO']
          if  path_info == '/'
            text = Tracks::Controller.render_default_root
          else
            text = "hello from tracks #{path_info}"
          end

          [200, {'Content-Type' => 'text/html'}, [text] ]
        end
      end
    end

That's it for now. Next time, we will create our own controllers, action, and dynamic pages using ERB.
