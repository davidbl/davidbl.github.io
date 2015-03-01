---
layout: post
title: Neo4j and Vagrant
---

Yeah, well,  I've been busy....


### TLDR;
  - don't try to install neo4j in the shared directory of a vagrant box
  - change the neo4j server config to allow access from the host machine
    - org.neo4j.server.webserver.address=0.0.0.0

### Full Story
I saw a twitter post a few days ago from [@analyticbridge](https://twitter.com/analyticbridge) about a [free book on graph databases](http://www.analyticbridge.com/group/books/forum/topics/free-o-reilly-book-graph-databases).
So, of course, I downloaded it and started reading.  Because it is hard to read a book about software without playing with the software, I fired up the MB Air, spun up a new vagrant box,
downloaded and un-tarred the latest version of [Neo4j community edition](http://neo4j.com/download/).

And then the trouble started.  I extracted the tar file in the /vagrant directory of my vagrant box and every time I tried to start the server I would get errors similar to:

````
Starting Neo4j Server...WARNING: not changing user
process [2460]... waiting for server to be ready........ Failed to start within 120 seconds.
````

The log file /NEO4J_HOME/data/log/console.log showed entries like...

````
2015-03-01 09:37:42.494+0000 INFO  [API] Setting startup timeout to: 120000ms based on -1
2015-03-01 09:37:45.368+0000 INFO  [API] Successfully started database
2015-03-01 09:38:50.857+0000 INFO  [API] Setting startup timeout to: 120000ms based on -1
Detected incorrectly shut down database, performing recovery..
2015-03-01 09:38:54.160+0000 INFO  [API] Successfully started database
2015-03-01 09:38:54.256+0000 INFO  [API] current RRDB is invalid, renamed it to /vagrant/neo4j-community-2.1.7/data/rrd-invalid-1425202734255
2015-03-01 09:42:50.103+0000 INFO  [API] Setting startup timeout to: 120000ms based on -1
Detected incorrectly shut down database, performing recovery..
````

Some googling around led me to believe that installing Neo4j in the shared directory (/vagrant) was the issue.
And, sure enough, extracting the file to another directory inside the vagrant box did the trick.  Neo4j now started with no issue.

But....
When I tried to hit the url for the web interface, localhost:7474,  I wasn't getting any data.  And yes, I had forwarded that port 
in my Vagrant file

````
  config.vm.network "forwarded_port", guest: 7474, host: 7474
````

So, even more googling around let me to this snippet on the [Neo4j server configuration page](http://neo4j.com/docs/stable/server-configuration.html)

````
#allow any client to connect
org.neo4j.server.webserver.address=0.0.0.0
````

Uncommenting that line in NEO4J_HOME/conf/neo4j-server.properties allowed me hit the browser-based UI
and start playing with Neo4j.

I hope maybe this post will save others a bit of googling.
