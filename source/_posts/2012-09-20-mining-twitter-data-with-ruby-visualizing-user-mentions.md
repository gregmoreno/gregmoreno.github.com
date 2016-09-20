---
layout: post
title: Mining Twitter Data with Ruby - Visualizing User Mentions
date: 2012-09-20 20:53:45.000000000 -07:00
categories:
- Coding
tags:
- graph
- mongodb
- ruby
- twitter
status: publish
type: post
published: true
author: 
---

In my <a href="/2012/09/05/mining-twitter-data-with-ruby-mongodb-and-map-reduce/">previous post on mining twitter data with ruby</a>, we laid our foundation for collecting and analyzing Twitter updates. We stored these updates in <a href="http://www.mongodb.org/">MongoDB</a> and used map-reduce to implement a simple counting of tweets.  In this post, we'll show relationships between users based on mentions inside the tweet.  Fortunately for us, there is no need to parse each tweet just to get a list of users mentioned in the tweet because Twitter provides the "entities.mentions" field that contains what we need. After we collected the "who mentions who", we then construct a directed graph to represent these relationships and convert them to an image so we can actually see it.

First, we start with the aggregation of mentions per user. We will use the same code base as last time. So if this is your first time, I recommend reading my previous related post or you can <a href="https://github.com/gregmoreno/tweetminer">follow the changes in Github</a>. Note to self:  Convert this to an actual gem in the next post.

    # user_mention.rb
    module UserMention

      def mentions_by_user
        map_command = %q{
          function() {
            var mentions = this.entities.user_mentions,
                users = [];
            if (mentions.length &gt; 0) {
              for(i in mentions) {
                users.push(mentions[i].id_str)
              }

              emit(this.user.id_str, { mentions: users });
            }
          }
        }

       reduce_command = %q{
          function(key, values) {
            var users = [];

            for(i in values) {
              users = users.concat(values[i].mentions);
            }

            return { mentions: users };
          }
        }

        options = {:out => {:inline => 1}, :raw => true, :limit => 50 }
        statuses.map_reduce(map_command, reduce_command, options)
     end

    end

We then again use map-reduce in MongoDB to implement our aggregation. Of course, this sort of thing can be done in Ruby directly but it would be way more efficient if we do it in MongoDB especially if you have a big collection to process.  Note that we limit the number of documents to process because we don't want our graph to look unrecognizable when we display it.

Now that we have our aggregation working, we construct a directed graph of user mentions using the <a href="http://rgl.rubyforge.org/rgl/index.html">rgl library</a>.

    require "bundler"
    Bundler.require
    require File.expand_path("../tweetminer", __FILE__)
    settings = YAML.load_file File.expand_path("../mongo.yml", __FILE__)
    miner = TweetMiner.new(settings)

    require "rgl/adjacency"
    require "rgl/dot"

    graph = RGL::DirectedAdjacencyGraph.new
    miner.mentions_by_user.fetch("results").each do |user|
      user.fetch("value").fetch("mentions").each do |mention|
        graph.add_edge(user.fetch("_id"), mention)
      end
    end

    # creates graph.dot, graph.png
    graph.write_to_graphic_file


Once you have the user-mentions relationships in a graph, you can do interesting things like who is connected to somebody and the degrees of separation. But for now, we are just interested in showing who mentioned whom.  Our sample program saves the graph to the file graph.dot (<a href="http://en.wikipedia.org/wiki/DOT_language">using the DOT language</a>) and PNG output. But the default PNG output is not laid out nicely. Instead, we will use the "<a href="http://www.graphviz.org/">neato</a>"  program to convert our graph.dot into a nice looking PNG file.

    $ neato -Tpng graph.dot -o mentions.png

When you view "mentions.png", you should see something similar as the one below. The labels are user IDs and the arrows show the mentioned users.

It would be cool to modify our program to use the users' avatars and also make it interactive. Or, use Twitter's streaming API and create an auto-update graph. I haven't done any research yet but I'm sure there is some Javascript library out there that can help us display graph relationships.
