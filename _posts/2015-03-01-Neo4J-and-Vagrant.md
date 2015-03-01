---
layout: post
title: Neo4j and Vagrant
---

Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.


Yeah, well,  I've been busy....


I saw a twitter post a few days ago from [@analyticbridge](https://twitter.com/analyticbridge) about a [free book on graph databases](http://www.analyticbridge.com/group/books/forum/topics/free-o-reilly-book-graph-databases).
So, of course, I downloaded it and started reading.  And its hard to read a book about software without playing with the software so I fired up the MB Air, spun up a new vagrant box,
downloaded and un-tarred the latest version of [Neo4j community edition](http://neo4j.com/download/).

And then the trouble starting.  I extracted the tar file in the /vagrant directory of my vagrant box and every time I tried to start the server I would get errors similar to:

````
Starting Neo4j Server...WARNING: not changing user
process [2460]... waiting for server to be ready........ Failed to start within 120 seconds.
````
