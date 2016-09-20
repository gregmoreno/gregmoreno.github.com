---
layout: post
title: How to create a wrapper gem for service APIs - part 1
date: 2012-06-07 00:41:01.000000000 -07:00
categories:
- Coding
tags:
- api
- gem
- ruby
status: publish
type: post
published: true
---

APIs are getting more and more popular as apps and services move to the cloud. Whenever you need to integrate a popular web service API into your Ruby app, 99.99% of the time there already exists a gem ready for use. That is a testament to how active Ruby developers are in supporting the community.

Even if integrating with the popular APIs is not in your radar, you may still have a need to create an API wrapper for internal use. For example, if your Rails application has grown tremendously (congratulations!), you may eventually need to adopt a services architecture to support upcoming features and make things manageable.

I created the <a href="https://github.com/gregmoreno/openamplify">gem for the Open Amplify API</a> as part of my exploration to data mining. When I first created it, my primary goal was simply to wrap the API. Though I still didn't write spaghetti code, it wasn't a good example of structured code either. Two years later (yep, that's how long I let the code rot), I decided to re-write the gem and adopt the architecture from the Twitter gem. It was a good exercise because not only I updated the gem for the newest API version, I also learned a great deal on how to write a gem.

### Setup the project

We will create a wrapper for the fictitious Awesome API and thus call our gem 'awesome'. To get things started, let's use bundler to set up our initial code.

    $> bundle gem awesome
    create awesome/Gemfile
    create awesome/Rakefile
    create awesome/LICENSE
    create awesome/README.md
    create awesome/.gitignore
    create awesome/awesome.gemspec
    create awesome/lib/awesome.rb
    create awesome/lib/awesome/version.rb
    Initializating git repo in /Users/greg/dev/code/awesome

This is the standard directory structure and naming convention of Ruby gems. The files Gemfile, Rakefile, and .gitignore are not necessary but they would be very useful while developing your gem.

### Gem dependencies

All gem dependencies should go into awesome.gemspec and not in Gemfile. Inside your Gemfile, the line 'gemspec' takes care of identifying the gems you needed in your local.

    $> more Gemfile
    source 'https://rubygems.org'

    # Specify your gem's dependencies in awesome.gemspec
    gemspec

### Versioning

You specify the version of your gem inside lib/awesome/version.rb

    $> more lib/awesome/version.rb
    module Awesome
      VERSION = "0.0.1"
    end

You may be wondering how is this used by the gem. Take a peek at awesome.gemspec and you'll see that Awesome::VERSION is used by the .gemspec file.

    $> more awesome.gemspec
    require File.expand_path('../lib/awesome/version', __FILE__)

    Gem::Specification.new do |gem|
    gem.authors = ["Greg Moreno"]
    gem.email = ["greg.moreno@gmail.com"]
    gem.description = %q{TODO: Write a gem description}
    gem.summary = %q{TODO: Write a gem summary}
    gem.homepage = ""

    gem.files = `git ls-files`.split($\)
    gem.executables = gem.files.grep(%r{^bin/}).map{ |f| File.basename(f) }
    gem.test_files = gem.files.grep(%r{^(test|spec|features)/})
    gem.name = "awesome"
    gem.require_paths = ["lib"]
    gem.version = Awesome::VERSION
    end

### Additional modules and classes 

You can write all your code in the file lib/awesome.rb and it would still work. However, in the spirit of making code maintainable, it is highly recommended that you put your classes and modules under the directory lib/awesome just like what we did with lib/awesome/version.rb

### Testing the gem

We will use minitest but you can always use any test framework you prefer. For our test setup, we need to do the ff:

* Setup our test directory manually since bundler didn't do this for us.
* Create a rake task to run our tests.
* Specify gem dependencies in our common test helper file.

And the steps and code below.

    $> mkdir test
    $> touch test/helper.rb
    $> mkdir test/awesome
    $> touch test/awesome/awesome_test.rb

    # Rakefile
    require 'bundler/gem_tasks'

    require 'rake/testtask'
    Rake::TestTask.new do |test|
      test.libs << 'lib' << 'test'
      test.ruby_opts << "-rubygems"
      test.pattern = 'test/**/*_test.rb'
      test.verbose = true
    end

    # test/helper.rb
    require 'awesome'
    require 'minitest/spec'
    require 'minitest/autorun'

Now that we have our testing in place, let's write a simple test and see if everything works.

    # test/awesome/awesome_test.rb
    require 'helper'

    describe Awesome do
      it 'should have a version' do
        Awesome::VERSION.wont_be_nil
      end
    end

    # Then, let's run the test
    $> rake test
    (in /Users/greg/dev/code/awesome)
    /Users/greg/.rbenv/versions/1.9.2-p290/bin/ruby -I"lib:lib:test" -rubygems "/Users/greg/.rbenv/versions/1.9.2-p290/lib/ruby/1.9.1/rake/rake_test_loader.rb" "test/awesome/awesome_test.rb"
    Loaded suite /Users/greg/.rbenv/versions/1.9.2-p290/lib/ruby/1.9.1/rake/rake_test_loader
    Started
    .
    Finished in 0.000695 seconds.

    1 tests, 1 assertions, 0 failures, 0 errors, 0 skips

    Test run options: --seed 55984

Perfect! Now, let's start working on the juicy parts of our gem.

### Configuration

Every webservice API would definitely require a per-user configuration and your gem should be able to support that. For example in the Twitter gem, some methods require authentication and you setup the default configuration with this:

    Twitter.configure do |config|
      config.consumer_key = YOUR_CONSUMER_KEY
      config.consumer_secret = YOUR_CONSUMER_SECRET
      config.oauth_token = YOUR_OAUTH_TOKEN
      config.oauth_token_secret = YOUR_OAUTH_TOKEN_SECRET
    end

Every API has different options but if you are wrapping a webservice, the options often fall into two categories - connections and functional options. For example,  connection-related options include the endpoint, user agent, and authentication keys while functional options include request format (e.g. json), number of pages to return and other parameters required by specific API functions. In some APIs, the  api key is passed as a parameter to GET calls so while it may be connection-related, it is better to group it with parameter options so you can easily encode all parameters in a single call.Our Awesome API is simple and will not deal with OAuth like the Twitter gem does. For the configuration, we should be able to do this:

    Awesome.api_key = 'YOUR_API_KEY'
    Awesome.format = :json
    # Other options are: user_agent, method

Now, let's write some tests. Of course, these should fail at first :)

    # test/awesome/configuration_test.rbrequire 'helper'

    describe 'configuration' do
      describe '.api_key' do
        it 'should return default key' do
          Awesome.api_key.must_equal Awesome::Configuration::DEFAULT_API_KEY
        end
      end

      describe '.format' do
        it 'should return default format' do
          Awesome.format.must_equal Awesome::Configuration::DEFAULT_FORMAT
        end
      end

      describe '.user_agent' do
        it 'should return default user agent' do
          Awesome.user_agent.must_equal Awesome::Configuration::DEFAULT_USER_AGENT
        end
      end

      describe '.method' do
        it 'should return default http method' do
          Awesome.method.must_equal Awesome::Configuration::DEFAULT_METHOD
        end
      end
    end

As I mentioned before, the best way to write your gem (or any program for that matter) is to cleary separate the functionalities into modules and classes. In our case, we will put all configuration defaults inside a module (i.e. lib/awesome/configuration.rb). We also want to provide class methods for the module Awesome which we can easily do using Ruby's 'extend'.

    # lib/awesome/configuration.rb

    module Awesome
      module Configuration
        VALID_CONNECTION_KEYS = [:endpoint, :user_agent, :method].freeze
        VALID_OPTIONS_KEYS = [:api_key, :format].freeze
        VALID_CONFIG_KEYS = VALID_CONNECTION_KEYS + VALID_OPTIONS_KEYS

        DEFAULT_ENDPOINT = 'http://awesome.dev/api'
        DEFAULT_METHOD = :get
        DEFAULT_USER_AGENT = "Awesome API Ruby Gem #{Awesome::VERSION}".freeze

        DEFAULT_API_KEY = nil
        DEFAULT_FORMAT = :json

        # Build accessor methods for every config options so we can do this, for example:
        # Awesome.format = :xml
        attr_accessor *VALID_CONFIG_KEYS

        # Make sure we have the default values set when we get 'extended'
        def self.extended(base)
          base.reset
        end

        def reset
          self.endpoint = DEFAULT_ENDPOINT
          self.method = DEFAULT_METHOD
          self.user_agent = DEFAULT_USER_AGENT

          self.api_key = DEFAULT_API_KEY
          self.format = DEFAULT_FORMAT
        end

      end # Configuration
    end

    # lib/awesome.rb
    require 'awesome/version'
    require 'awesome/configuration'

    module Awesome
      extend Configuration
    end

    $> rake test
    (in /Users/greg/dev/code/awesome)
    /Users/greg/.rbenv/versions/1.9.2-p290/bin/ruby -I"lib:lib:test" -rubygems "/Users/greg/.rbenv/versions/1.9.2-p290/lib/ruby/1.9.1/rake/rake_test_loader.rb" "test/awesome/awesome_test.rb" "test/awesome/configuration_test.rb"
    Loaded suite /Users/greg/.rbenv/versions/1.9.2-p290/lib/ruby/1.9.1/rake/rake_test_loader
    Started
    .....
    Finished in 0.001600 seconds.

    5 tests, 5 assertions, 0 failures, 0 errors, 0 skips

Our gem will not be awesome if we don't support a 'configure' block like what the Twitter gem does. We want to setup the configuration like this:

    Awesome.configure do |config|
      config.api_key = 'YOUR_API_KEY'
      config.method = :post
      config.format = :json
    end

Fortunately, it's an easy fix. We just need to add a 'configure' method to the Configuration module. We also update our tests to make sure this new method works.

    # lib/awesome/configuration.rb
    def configure
      yield self
    end

    # test/awesome/configuration_test.rb
    after do
      Awesome.reset
    end

    describe '.configure' do
      Awesome::Configuration::VALID_CONFIG_KEYS.each do |key|
        it "should set the #{key}" do
          Awesome.configure do |config|
            config.send("#{key}=", key)
            Awesome.send(key).must_equal key
          end
        end
      end
    end

Before we move on, let's take a second look at our configuration tests. We have tests for checking default values and setting-up new ones.  What if we added a new configuration key for our gem? The 'configure' tests will be able to handle the new key but we still have to add another test for checking the default value. And we don't want to right another test code, right?  More importantly, we don't want our tests to yield false positives. If we fail to add the 'default value' check, our tests will still pass even though  we forgot to set a default value.Let us remove all our default value tests and replace it with code that relies on VALID_CONFIG_KEYS instead.

    # test/awesome/configuration_test.rb
    Awesome::Configuration::VALID_CONFIG_KEYS.each do |key|
      describe ".#{key}" do
        it 'should return the default value' do
          Awesome.send(key).must_equal Awesome::Configuration.const_get("DEFAULT_#{key.upcase}")
        end
      end
    end

    $> rake test
    (in /Users/greg/dev/code/awesome)
    /Users/greg/.rbenv/versions/1.9.2-p290/bin/ruby -I"lib:lib:test" -rubygems "/Users/greg/.rbenv/versions/1.9.2-p290/lib/ruby/1.9.1/rake/rake_test_loader.rb" "test/awesome/awesome_test.rb" "test/awesome/configuration_test.rb"
    Loaded suite /Users/greg/.rbenv/versions/1.9.2-p290/lib/ruby/1.9.1/rake/rake_test_loader
    Started
    ...........
    Finished in 0.002935 seconds.

    11 tests, 11 assertions, 0 failures, 0 errors, 0 skips

    Test run options: --seed 21540

### Configuring clients

Our end goal is to wrap API calls that fits nicely into our application and the common approach to do that is to wrap the API calls under a 'Client' class. Depending on the size of the API you want to support, the Client class maybe delegating the method calls to other classes and modules but from the point of view your program, the action happens inside the Client class. There are two ways to configure the Client class:

* It inherits the configuration values defined in the Awesome module;
* It overrides the configuration values per client

    # Use the values defined in the Awesome module
    client = Awesome::Client.new
    client.make_me_awesome('gregmoreno')

    client_xml = Awesome::Client.new :format => :xml
    client_json = Awesome::Client.new :format => :json

We are not going to show our tests in here but if you are interested, you can view the <a href="https://github.com/gregmoreno/awesome/blob/master/test/awesome/client_test.rb">test code from the github</a> repository. Instead, we show the code that handles the two scenarios for client configuration.

    # lib/awesome/client.rb

    module Awesome
      class Client

        # Define the same set of accessors as the Awesome module
        attr_accessor *Configuration::VALID_CONFIG_KEYS

        def initialize(options={})
          # Merge the config values from the module and those passed
          # to the client.
          merged_options = Awesome.options.merge(options)

          # Copy the merged values to this client and ignore those
          # not part of our configuration
          Configuration::VALID_CONFIG_KEYS.each do |key|
            send("#{key}=", merged_options[key])
          end
        end

      end # Client
    end

We also need to update our Awesome module. First, we need to require the new file  awesome/client.rb so it will be loaded when we require the gem.  Second, we need to implement a method that returns all the configuration values inside the Awesome module. Since this is still about configuration, our new method should go inside the Configuration module.

    # lib/awesome/configuration.rb
    def options
      Hash[ * VALID_CONFIG_KEYS.map { |key| [key, send(key)] }.flatten ]
    end

We're finally done with the configuration part of our gem. I know it's a lot of work for a simple task but we managed to put a good structure in our code. Plus, we learned how to make our tests less brittle, and use Ruby's awesome power to make our code better.   In our next installment, we'll discuss requests and error handling.
