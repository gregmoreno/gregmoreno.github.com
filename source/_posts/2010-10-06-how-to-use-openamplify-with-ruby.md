---
layout: post
title: How to use OpenAmplify with Ruby
date: 2010-10-06 02:41:45.000000000 -07:00
categories:
- Coding
tags:
- ruby
status: publish
type: post
published: true
---

The [OpenAmplify API](http://community.openamplify.com/blogs/quickstart/pages/overview.aspx) reads text you supply and returns linguistic data explaining and classifying the content. What you do with that analysis is, in the fine tradition of APIs and mashups, up to you. Some possibilities might include pairing ads with articles, creating rich tag-clouds, or monitoring the tone of forum threads.

I created a ruby gem to simplify the use of the OpenAmplify API. It’s still in the early stages but should be enough to get you started.

In case you need a different format, OpenAmplify supports XML, JSON, RDF, CSV. It can also return the result as a fancy HTML page.

The source code is available in github [http://github.com/gregmoreno/openamplify](http://github.com/gregmoreno/openamplify)

