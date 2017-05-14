---
layout: post
title: "Random wierd Cassandra issue"
category: cluster
tags: [cluster, io, programming, hacks, datetime]
author: "michael"
---

### So this happened today

I have been working with Cassandra recently and I enjoy the idea of it. I think it has potential even if I do need to do a lot of tuning for our cluster. Right now it is set up to run on five nodes out of a dedicated raid array on each node we are using for testing such things. It has a lot of space and nobody else is using it so it should be really nice.

Initially I set it up to use the normal private cat5 network just to run some tests to see if it would work, etc, could be interoperable with Apache Spark in our setup that sort of thing. Everything was, if a bit slower than expected. So I stopped it, moved everything to our infiniband network, and started everything back up and thats when I began to notice some really bizarre behavior. One node out of the five, at startup, could see all the other nodes but then would soon stop seeing one specific node. This happened consistently and nothing I could find seemed to address the issue. I checked and rechecked the setup and everything was fine. My feeling is that it was like this before which probably contributed to it being slow (it feels faster now that I have fixed it).

I don't know why yet, but the solution was really REALLY strange. All the nodes had system times that were relatively close to each other with the exception of the one that was randomly being reported as unreachable which was about a half hour behind the others. The wierd thing is that only one node reported it as unreachable. The fix was just to set the slow node's system time ahead to more closely match the other nodes. Since then the problem has not resurfaced. Very VERY strange, and it wasted most of my morning.

(Yes I know they should be running ntp, its on the to-do list. I don't know why they aren't already running it. The entire cluster is due for a massive system overhaul where we wipe all the nodes and do a full system reinstall so I will address it then.)