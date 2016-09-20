---
layout: post
title: Create your own Rails 3 engine
date: 2012-05-29 22:47:53.000000000 -07:00
categories:
- Coding
tags:
- engine
- rails
- ruby
status: publish
type: post
published: true
---
Engine is an interesting feature of Rails. Engines are miniature applications that live inside your application and they have structure that you would normally find in a typical Rails application. If you have used the Devise gem, which itself is an engine, you know the benefits of being able to add functionality to your application with just a few lines of code. Another great benefit of engines is when you or your team are maintaining a number of applications the common functionalities can be extracted into engines.

Engines are already available prior to Rails 3 but it is not a core feature of the framework. As such, engine developers resorted to monkey-patching which, oftentimes, lead to engines breaking when Rails gets updated. In Rails 3.1, engines are now supported by the framework and there is now a clealy defined place where to hook your engines into Rails.

Now, let us go through the steps of building a simple engine. We will be working on authentication engine (like Devise) that allows users of your application to use their Twitter or Facebook credentials.

    # This is the app that will use our engine
    $> rails new social_app

    # This is our engine
    $> rails plugin new undevise --mountable

The –mountable option tells Rails you want to generate a mountable plugin, commonly known as engine. When you look at the directory structure of your engine, it is much different from your Rails app. The engine has controllers, models, views, mailers, lib, and even its own config/routes.rb (didn’t we just said it is a miniature Rails app).

### Include the engine in your app

Just like any gem, you should update your app’s Gemfile to use the engine we created.

    # social_app/Gemfile
    gem 'undevise', :path => '../undevise'

Of course, you can set :path to any location or if it is in a git repository, you can use the :git option. If you are developing your engine alongside your app, a better approach is to use gem groups in your Gemfile. For example:

    group :development do
      gem 'undevise', :path => '../undevise'
    end

    group :production do
      gem 'undevise', :git => 'git://github.com/yourname/undevise.git'
    end

After adding the engine in your Gemfile, let’s make sure all dependencies are available for the application. If everything works, you should be able to see a reference to undevise inside Gemfile.lock

    $> cd social_app
    $> bundle install
    $> more Gemfile.lock
    PATH
      remote: ../undevise
      specs:
        undevise (0.0.1)
          rails (~> 3.2.1)

### Mount the engine

Next, we will mount the engine and see if we can route requests to it. What this does is make sure requests starting with /auth will be passed to our engine.

    # social_app/config/routes.rb
    SocialApp::Application.routes.draw do
      mount Undevise::Engine, :at => '/auth'
    end

    # Run the social app. Make sure you are in the social_app directory.
    $> rails s

    # Then visit http://localhost:3000/auth

When you visit ‘/auth‘, you will get a routing error because you haven’t defined any routes in your engine yet.

    # undevise/config/routes.rb
    Undevise::Engine.routes.draw do
      root :to => 'auth#index'
    end

Remember even though your engine is mounted at ‘/auth’, what your engine sees is the path after the ‘/auth’. Routes in engines are namespaced to avoid conflicts with your app. You can change the mounted path in your Rails app anytime and your engine wouldn’t care. Let’s try again and see what Rails would tell us.

    $> cd social_app
    $> rails s

Perfect! Now we know the request is being passed to our engine. We now just have to define our controller.

    $ cd undevise
    $ rails g controller auth

    # undevise/app/controllers/undevise/auth_controller.rb
    module Undevise
      class AuthController < ApplicationController
        def index
          render :text => 'Hello world'
        end

      end
    end

    # Now, visit http://localhost:3000/auth


Cool! We have the obligatory hello world program working. At the the risk of sounding like a broken record, remember your engine code should be namespaced. If you forget this, strange things will happen to your application and Rails will not usually complain about it.

### Gem dependencies

I’m sure your idea for an engine is very far from what we have shown so far. When you generate an engine, it also creates a .gemspec file. While in your Rails app you list the gems in Gemfile, in your engine you list the gems inside the .gemspec file. This can be confusing because the engine also contains a Gemfile.

    $> cd undevise
    $> more Gemfile

    source "http://rubygems.org"

    # Declare your gem's dependencies in undevise.gemspec.
    # Bundler will treat runtime dependencies like base dependencies, and
    # development dependencies will be added by default to the :development group.
    gemspec

As you can see, there is no need to list the gems your engine needs in the Gemfile. The line ‘gemspec’ makes sure the gems you listed in your .gemspec file are installed when you run bundle install.

Now, let’s add some gems in our engine and see how it will affect our Rails app.

    # undevise/undevise.gemspec
    s.add_dependency "rails", "~> 3.2.1"
    s.add_dependency "omniauth"
    s.add_dependency "omniauth-twitter"
    s.add_dependency "omniauth-facebook"


    $ cd social_app
    $ bundle install
    $ more Gemfile.lock
    PATH
      remote: ../undevise
      specs:
        undevise (0.0.1)
          omniauth
          omniauth-facebook
          omniauth-twitter
          rails (~> 3.2.1)

Here we can see the gems we specifed in undevise/undevise.gemspec are also included in the main Rails app.

### Configure OmniAuth

If you are using omniauth directly in your app, your will configuration will definitely be in the file config/initializers/omniauth.rb. Since our engine is pretty much just another Rails app, it will also have its own config/initializers/omniauth.rb file. The only consideration with regards to the configuration is where would the Twitter or Facebook credentials be located. You definitely don’t want to embed it in your engine.

Our solution is to store the credentials inside a config/twitter.yml file (or config/facebook.yml) inside your main Rails app. Then have our engine pull the values out of these files to configure omniauth.

    $> cd undevise/config
    $> mkdir initializers
    $> touch initializers/omniauth.rb

    # We have to create the initializers directory because it not created by default.

    # undevise/config/initializers/omniauth.rb
    providers = %w(twitter facebook).inject([]) do |providers, provider|
      fpath = Rails.root.join('config', "#{provider}.yml")

      if File.exists?(fpath)
        config = YAML.load_file(fpath)
        providers << [ provider, config['consumer_key'], config['consumer_secret'] ]
      end

      providers
    end

    raise 'You have not created config/twitter.yml or config/facebook.yml' if providers.empty?

    Rails.application.config.middleware.use OmniAuth::Builder do
      providers.each do |p|
        provider *p
      end
    end

Now, let’s go back to our main Rails app and start the server.

    $> cd social_app
    $> rails s
    => Booting WEBrick
    => Rails 3.2.1 application starting in development on http://0.0.0.0:3000
    => Call with -d to detach
    => Ctrl-C to shutdown server
    Exiting
    /Users/greg/dev/tmp/ruby/engine-tutorial/undevise/config/initializers/omniauth.rb:12:in `<top (required)>': You have not created config/twitter.yml or config/facebook.yml (RuntimeError)

Oops! We forgot to create our Twitter or Facebook configuration file. In your main Rails app, go ahead and create config/twitter.yml. If you are not familiar with Twitter apps, visit their developer site at https://dev.twitter.com/

### Gemfile dependencies and sub-depencies

    # social_app/config/twitter.yml
    consumer_key:  'APP_CONSUMER_KEY'
    consumer_secret: 'APP_CONSUMER_SECRET'

    $> rails s
    => Booting WEBrick
    => Rails 3.2.1 application starting in development on http://0.0.0.0:3000
    => Call with -d to detach
    => Ctrl-C to shutdown server
    Exiting
    /Users/greg/dev/tmp/ruby/engine-tutorial/undevise/config/initializers/omniauth.rb:12:in `<top (required)>': uninitialized constant OmniAuth (NameError)
         from /Users/greg/.rbenv/versions/1.9.3-p0/lib/ruby/gems/1.9.1/gems/railties-3.2.1/lib/rails/engine.rb:588:in `block (2 levels) in <class:Engine>'

The NameError occurs because during the main Rails’ app boot up, Bundler will only require dependencies listed in the Gemfile but not the sub-dependencies. As you can see from Gemfile.lock, omniauth is not a direct dependency. You could list the gems in your main app’s Gemfile but that’s defeating the purpose of isolating the gem dependencies through your engine.

The solution right now is to require your dependencies inside your engine and the place to do that is inside lib/undevise/engine.rb

    # undevise/lib/undevise/engine.rb
    require 'omniauth'
    require 'omniauth-twitter'
    require 'omniauth-facebook'

    module Undevise
      class Engine < ::Rails::Engine
        isolate_namespace Undevise
      end
    end

After listing required dependencies inside your engine, restart your main Rails app, then visit http://localhost:3000/auth/twitter/

When you visit http://localhost:3000/auth/twitter/, you should see the error above. The callback url is part of OmniAuth’s behaviour and should be fixed by adding a route in your engine and adding the method to handle it in your controller.

    # undevise/config/routes.rb
    Undevise::Engine.routes.draw do
      root :to => 'auth#index'
      match ':provider/callback' => 'auth#callback'
    end

    # undevise/app/controllers/undevise/auth_controller.rb
    module Undevise
      class AuthController < ApplicationController

        def index
          render :text => 'Hello world'
        end

        def callback
          render :text => "Hello from #{params[:provider]}"
        end

      end
    end

If everything work fine, you should see a message from Twitter.

We only scratched the surface with Rails 3 engine. Your engine, much like any normal Rails app, can have models and migrations, javascripts, css, specs, etc. If you want to dig deeper into engines, I recommend Rails 3 in Action by Ryan Bigg and Yehuda Katz. It includes a whole chapter about engines, discussion of middleware, and how tests your engine.
