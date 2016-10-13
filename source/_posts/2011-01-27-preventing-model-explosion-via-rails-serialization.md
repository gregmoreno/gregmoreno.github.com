---
layout: post
title: Preventing model explosion via Rails serialization
date: 2011-01-27 02:33:25.000000000 -08:00
categories:
- Coding
tags:
- rails
status: publish
type: post
published: true
---

A great thing about ActiveRecord is you can easily add a new model to your application and play around with it as you progress. However, this power can easily be overused leading to unnecessary overhead in your code.

Consider the case where you have preferences for each user. For example, a user may opt to show or hide his email address, adjust his timezone, or language. One solution is to simply add new columns to the users table that correspond to each preference type. For example, you can have a ‘show_email’, ‘timezone’, ‘locale’ columns in the ‘users’ table, which can make your table become wide as you add more preferences options. Another option is to use a separate ‘preferences’ table.

    class User < ActiveRecord::Base
      has_many :preferences
    end

    class Preferences < ActiveRecord::Base
      belongs_to :user

      # name  - preference name
      # value - preference value
    end

Note there is no user interface to add or remove ‘preferences’, i.e. the kinds of preferences are fixed. Of course, in the future you may add a new kind of preference but this kind of work is better done outside of the user interface. Since that is the case, there is no need to represent ‘preferences’ as a separate model.

One better alternative is to use Rails serialization to store the different kinds and user-specific values. The code would look like this:

    class User < ActiveRecord::Base
      serialize :preferences, Hash
    end

    u = User.new
    u.preferences = {:show_email => true, :locale => :en }
    u.save

    # somewhere in your view using haml
    - if @user.preferences[:show_email]
      = @user.email

Using ‘serialize’ results in less code, fewer tables, and less overall complexity. However, with serialization you lose the ability to efficiently search the preferences data. The million-dollar question is do you need to query these preferences? Do you need a finder that returns all users who wants to show their email?

One issue I had with ‘serialize’ is that by using it, I expose the implementation details. In the display example above, it is obvious I had it stored as a Hash. I would rather hide this detail and present the preferences attributes as user attributes instead. I also want default values for every user.

For example:

    u = User.new  # automatically assigns the default preferences
    u.preferences
    => {:show_email => false, :locale => :en}

    u.show_email = true  # I can change it like an attribute via @user.update_attributes(params[:user])
    u.preferences
    => {:show_email => true, :locale => :en}

I have created a module to support this. It is not a unique problem so others may have probably released a gem or plugin to do this. (I actually never bothered to search for one.) Nevertheless, it was a good exercise in metaprogramming.

To use my implementation, simply call ‘serializeable’ with the column you want to serialize and the default values.

    class User < ActiveRecord::Base
      serializeable :preferences, :show_email => true, :locale => :en
    end

Below is the implementation of ‘serializeable’. The convention is to save it under your ‘lib’ folder and include it in your ‘config/application.rb’ if you are using Rails 3.

    module AttributeSerializer
      module ActiveRecordExtensions
        module ClassMethods

          def serializeable(serialized, serialized_accessors={})  
            serialize serialized, serialized_accessors.class

            serialized_attr_accessor serialized, serialized_accessors
            default_serialized_attr serialized,  serialized_accessors
          end

          # Creates the accessors
          def serialized_attr_accessor(serialized, accessors)
            accessors.keys.each do |k|
              define_method("#{k}") do
                self[serialized] && self[serialized][k]
              end

              define_method("#{k}=") do |value|
                self[serialized][k] = value
              end
            end
          end

          # Sets the default value of the serialized field
          def default_serialized_attr(serialized, accessors)
            method_name =  "set_default_#{serialized}"
            after_initialize method_name 

            define_method(method_name) do
              self[serialized] = accessors if self[serialized].nil?
            end
          end

        end
      end
    end

    class ActiveRecord::Base
      extend AttributeSerializer::ActiveRecordExtensions::ClassMethods
    end


ActiveRecord is both easy and powerful. It can also lead to misuse and abuse. Even though you are adding just one model, remember that it is not just the model class itself. You are also adding the database migrations, unit tests, factories, finders, and validations that go along with the model. Next time you have a new requirement, see if serialization can do a better job.

Update: Adam Cuppy converted this code into a [Rails plugin](https://github.com/DYE/has_serialized) while [Jay added dynamic finder methods](https://gist.github.com/842797). I also moved this into a gem I called [fancy_serializer](https://github.com/gregmoreno/fancy_serializer).

