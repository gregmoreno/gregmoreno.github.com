---
layout: post
title: 'Ruby 101: Improving your code by defining methods dynamically'
date: 2011-01-03 02:39:06.000000000 -08:00
categories:
- Coding
tags:
- ruby
status: publish
type: post
published: true
meta:
---

Let’s say you have a user and you want to check its role.

    class User
      attr_accessor :role
    end

    u = User.new
    u.role = 'admin'

    # somewhere in your code you check the role

    if u.role == 'admin'
      puts 'admin'
    elsif u.role == 'moderator'
      puts 'moderator'
    elsif u.role == 'guest'
      puts 'guest'
    end

Using a string value is bad code and you can improve this by using constants instead. But still, this is bad code becauses it exposes implementation details of your User class.

For our first improvement, we define methods that check the user’s role and hide the implementation of the role checking inside the User class.

    class User
      attr_accessor :role

      def is_admin?
        self.role == 'admin'
      end

      def is_moderator?
        self.role == 'moderator'
      end

      def is_guest?
        self.role == 'guest'
      end

    end

    u = User.new
    u.role = 'guest'

    if u.is_admin?
      puts 'admin'
    elsif u.is_moderator?
      puts 'moderator'
    elsif u.is_guest?
      puts 'guest'
    end

Our first improvement is definitely better than the original but there are duplicate code in the role checking. You can eliminate the duplicate code by delegating the role checking to a single method.

    class User
      attr_accessor :role

      def is_admin?
        is_role? 'admin'
      end

      def is_moderator?
        is_role? 'moderator'
      end

      def is_guest?
        is_role? 'guest'
      end

      protected

      def is_role?(name)
        self.role == name
      end

    end

Our second improvement is a classic refactoring technique and common in any modern programming language. In other words, there is nothing “Ruby” about it. Before you get bored, I will now show the Ruby version.

The Ruby version uses `#define_method` to further eliminate duplicate code.

    class User
      attr_accessor :role

      def self.has_role(name)
        define_method("is_#{name}?") do
          self.role == "#{name}"
        end
      end

      has_role :admin
      has_role :moderator
      has_role :guest

    end

By using `#define_method`, we were able to add instance methods to our class User. You can check the new instance methods via irb.

    ruby-1.9.2-p0 > User.instance_methods.grep /^is/
    => [:is_admin?, :is_moderator?, :is_guest?, :is_a?]

Note that `#has_role` is just another method and as such you can modify it to accept several parameters, an array, or other class. For example, we can make ‘has_role’ accept a list of roles.

    class User
      attr_accessor :role

      def self.has_roles(*names)
        names.each do |name|
          define_method("is_#{name}?") do
            self.role == "#{name}"
          end
        end
      end

      has_roles :admin, :moderator, :guest
    end
