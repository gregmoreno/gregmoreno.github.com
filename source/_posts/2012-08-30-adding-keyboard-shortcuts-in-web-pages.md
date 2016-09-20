---
layout: post
title: Adding keyboard shortcuts in web pages
date: 2012-08-30 20:52:53.000000000 -07:00
categories:
- Coding
tags:
- coffeescript
- ruby
- sinatra
- vim
status: publish
type: post
published: true
---

Adding keyboard shortcuts to interact with your web pages seems like a useless feature when the rest of the world is using a mouse. But for a programmer who wants everything to be a few keystrokes away, keyboard shortcuts are very handy. 

In this tutorial, we will add a simple scrolling shortcuts to our webpage.  This is just to illustrate what is possible. So please, do not copy-and-paste this to your production code.

### What do we need?

* <a href="http://jquery.com/">jquery</a>
* <a href="http://www.sinatrarb.com/">sinatra</a>
* <a href="http://coffeescript.org/">coffeescript</a>

Actually, the only critical piece we need is jQuery and knowledge of Javascript. However, since I am more of a Ruby guy, we will use Sinatra to build the page and CoffeeScript to write the Javascript.

### Build the pages

The screenshot below (left side) shows how our directory structure would look like. It is pretty much a standard Sinatra structure.

Our HTML page displays 10 entries where each is grouped under a "div" element with an  ".entry" class and an  ID.  We also add in some styling in our page to distinguish each entry.

    <!DOCTYPE HTML>
    <html>
    <head>
      <meta http-equiv="content-type" content="text/html; charset=utf-8" />
      <title>Index</title>
      <link rel="stylesheet" href="css/style.css"/> <script type="text/javascript" charset="utf-8" src="http://code.jquery.com/jquery-1.7.1.min.js"></script>
      <script type="text/javascript" charset="utf-8" src="js/app.js">
      </script>
    </head>
    <body>
      <% 1.upto(10) do |i| %>
        <div id="<%= "entry_#{i}" %>"class="entry">
          <%= "Title #{i}" %>
          <p>Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.</p>
        </div>
      <% end %>
    </body>
    </html>

If everything is setup correctly, you should be able to run the app and see 10 entries.

    $ ruby app.rb
    [2012-08-30 13:48:44] INFO WEBrick 1.3.1
    [2012-08-30 13:48:44] INFO ruby 1.9.2 (2012-04-20) [x86_64-darwin12.1.0]
    == Sinatra/1.3.3 has taken the stage on 4567 for development with backup from WEBrick
    [2012-08-30 13:48:44] INFO WEBrick::HTTPServer#start: pid=12415 port=4567

Now for the juicy part. When the user presses 'j', we will scroll to the next entry while 'k' scrolls to the previous. If you are a Vim user, you know why.

    current_entry = -1

    $(document).keydown (e) ->
      switch(e.keyCode)
      when 74 then scroll_to_next() # j
      when 75 then scroll_to_previous() # k

    scroll_to_next = ->
      #alert "scroll to next"
      current_entry++
      scroll_to_entry(current_entry)

    scroll_to_previous = ->
      if current_entry > 0
        current_entry–
      scroll_to_entry(current_entry)

    scroll_to_entry = (entry) ->
      # Get the element we need to scroll to
      id = $(".entry")[entry].id
      $("html, body").animate { scrollTop: $("##{id}").offset().top }, "slow"

That's it! As I've mentioned before, this is not production ready. For example, the shortcut should not interfere with other actions in your page like when the user is interacting with an input field. This also assumes the current visible entry is the first one.

This post is based from the book <a href="http://webdevelopmentrecipes.com/">Web Development Recipes</a>. If you are looking for quick reference on how to improve your project, I suggest reading the book.

