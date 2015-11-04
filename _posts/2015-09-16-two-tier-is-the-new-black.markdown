---
layout:     post
title:      "Two Tier Is The New Black"
subtitle:   "Firebase Plus Smart Phones = Who Needs A Server?"
date:       2015-09-16 12:00:00
author:     "Hays"
header-img: "img/noServers.png"

---

# Confession of a Backend Dev #
For a number of years now, I have written back end server code. From server side templating with rails to 
microservice REST apis with [Liberator](http://clojure-liberator.github.io/liberator/) (Liberator is really 
just a clojure version of [Webmachine](https://github.com/webmachine/webmachine/wiki/Overview) which is nice).
My latest project has shown me a new way though: No middle tier.

# No Middle Tier? How does that work?? #
It's like back to the future. When I first started, it was the tail end of the two tier architectures. Things like an
Oracle Forms, Delphi, or even VisualBasic fat frontends would go straight to a database without anything in the middle.
Obviously, the web changed that with dynamic templating on the server and everything that followed. Now, I am seeing, and experiencing,
something more like the old two tier architectures in places. Think [Parse](https://www.parse.com/) or [Firebase](https://www.firebase.com/).
In essence they are really the database and the app (especially mobile apps) have become the fat client.

# Why create a bunch of apis when you can just connect to the database? #
It seems to me that a lot of the API development these days is just serving json data blobs. Authenticate, query, and return the data. Sure you
can do some processing there. It's how it has been done for a while. But why? Why do that if you can just move that logic to the app itself? Who
wants to manage and support a bunch of web api servers if you don't have to? I am coming to the conclusion that you shouldn't.

# Don't Misunderstand #
While I am advocating the new model of two tiers, don't misunderstand me by thinking I am advocating no backend servers for anything. I do think
that analytics and soft realtime data analysis are becoming even more critical. So that whole area will continue without a doubt. Also, exposing APIs, or 
think your database, does have some place for integration between organizations. There are certainly good cases to expose functionality to others, but one should
really question whether it is necessary.

![Three To Two](/img/ThreeToTwo.png)

# Analytics Is Just Another DB Client #
It's special certainly, but don't make it more than it needs to be.

# Two Tier Benefits #
+ One codebase for the app
+ Lots of compute power in smart phones these days
+ No deploy/manage web tier servers
    This is really the crux of this post. Anything you can get rid of reasonably you should.
+ Spend more time on the value add. The app and the analytics.
+ If you can get to a place where an EC2 deployment is a pain point to get rid of, you are probably in a good spot!

