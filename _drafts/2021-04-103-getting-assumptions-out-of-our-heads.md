---
layout: post
comments: true
author: Justin Pietsch
title: Getting Assumptions out of our Heads
excerpt: 
description: Too much of networking requires network engineers to keep a lot in their heads. We need to get that out.
---

Some of the best network engineers I've experienced can keep a lot in their heads. Maybe it's all the OSPF state machines as well as all the LSAs and what is in them. Or when they are debugging and they run various show commands they can remember which IP was the neighbor that they need to care about. I can't do that. If I have to run a show command that has IP addresses that I need to resolve to know who the neighbors are, I'm in trouble. 

But even if you are goo at that, I think it's a very bad habit to rely on that. It appears to me that a lot of networking depends on engineers and operators keeping everything in their head. They aren't in the habit of getting ideas, concepts, etc out of their heads. This is bad, because when they are in your head, they are unexamined, and as things get more complicated you need to understanding your assumptions and examine them.

**Engineering is the art of tradeoffs.** When making a decision, or even when operating on that decision, you have a set of assumptions about how those things will work. When you don't know what your assumptions are, you can't know that you are making tradeoffs. In real engineering disciplines, those assumptions can turn into numbers, models, and equations. In software,  you get it out in code, and you test it through tests. In networking people are rewarded for keeping all those assumptions in their heads. I want it out. 

I'm much more interested in figuring out how to do Assumption Based Networking than Intent Based Networking. Well, I don't think you can do IBN without recording your assumptions first. If you don't record your assumptions, then you are using somebody else's. And the recording of assumptions is critical.


## So What does this mean?
Let's say I have a core network with 2 routers. The (obvious) assumption is that I can afford for only one of those routers to go down at any one time. I can only afford to have < then 50% capacity on those routers, because one might fail and I need to be able to support all traffic with a single failure. Sounds good. This means I do things like have capacity alarms at 50%. But mostly, because it's so obvious, there are a bunch of rules I have in my head that I don't even think about. If one of the routers is down, I can't do maintenance on the other. If I am close to 50% utilization, I also need to make sure that all the interfaces on both routers are up all the time. But none of these rules are written down. So as the network changes and as I add automation

Now I'm upgrading to 4 routers. What are my assumptions


### Examples from he past
One example I had for a long time was that you need to buy big chassis routers for datacenter networks. You need lots of interfaces and they have all the fancy and sophisticated switch backplanes to get you the performance that you need. But why do you need that? It turns out you don't.