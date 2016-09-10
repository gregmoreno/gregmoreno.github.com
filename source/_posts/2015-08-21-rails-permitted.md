---
layout: post
title: 'Rails #permitted'
date: 2015-08-21
categories:
- Coding
tags: []
status: publish
type: post
published: true
---

I recently upgraded a personal app I use for learning new things in Rails. But when I upgraded from
4.1.4 to 4.1.12 I encountered this familiar error.

    Customer.where(auth.slice(:provider, :uid)).first_or_initialize
    ActiveModel::ForbiddenAttributesError: ActiveModel::ForbiddenAttributesError
    from /Users/greg/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/activemodel-4.1.12/lib/active_model/forbidden_attributes_protection.rb:21:in `sanitize_for_mass_assignment`

Now I remember. It's one of those mass assignments where you have to specify the 'permitted' values before you can continue. In Rails 4, it's good practice
to whitelist the attributes you receive in the controller and it goes something like this:

    def user_params
      params.require(:user).permit(:username, :email, :password)
    end

    # somewhere in the controller
    Customer.create(user_params)

Now, let's use this idiom. It should be easy, right?

    > auth.permit(:provider, :uid)
    => nil

Wait, that didn't go as expected. How about just simply composing the hash?

    > Customer.where(provider: auth[:provider], uid: auth[:uid]).first_or_initialize
    => #<Customer:0x007fd9168ffa88>

Interesting. #permit returns nil, using plain hash works, and #slice doesn't.

    > auth.slice(:provider, :uid).class
    => OmniAuth::AuthHash < Hashie::Mash

It shouldn't matter what auth is as long as it behaves like what the Customer model expects. But what
does the Customer model expects? Actually, the error message is telling us what it expects. In Rails,
there is this module for mass assignment protection:

    # https://github.com/rails/rails/blob/master/activemodel/lib/active_model/forbidden_attributes_protection.rb
    module ActiveModel
      # Raised when forbidden attributes are used for mass assignment.
      #
      #   class Person < ActiveRecord::Base
      #   end
      #
      #   params = ActionController::Parameters.new(name: 'Bob')
      #   Person.new(params)
      #   # => ActiveModel::ForbiddenAttributesError
      #
      #   params.permit!
      #   Person.new(params)
      #   # => #<Person id: nil, name: "Bob">
      class ForbiddenAttributesError < StandardError
      end

      module ForbiddenAttributesProtection # :nodoc:
        protected
          def sanitize_for_mass_assignment(attributes)
            if attributes.respond_to?(:permitted?) && !attributes.permitted?
              raise ActiveModel::ForbiddenAttributesError
            else
              attributes
            end
          end
          alias :sanitize_forbidden_attributes :sanitize_for_mass_assignment
      end
    end

Nothing fancy here. Rails does a simple check whether to allow mass assignment or not.

    > auth.slice(:provider, :uid).permitted?
    => false

    > { provider: auth[:provider], uid: auth[:uid] }.permitted?
    NoMethodError: undefined method `permitted?' for {:provider=>"facebook", :uid=>"123"}:Hash

OmniAuth::AuthHash does not even allow it. Plain Hash works because it doesn't even respond to #permitted.
