---
layout: post
comments: true
author: Justin Pietsch
title: Getting Assumptions out of our Heads
excerpt: 
description: Too much of networking requires network engineers to keep a lot in their heads. We need to get that out.
---

Networking requires a lot of decisions by many different people. Every point of designing, building, scaling, operating networks requires decisions. Many of those decisions have even more assumptions behind them.

Some of the best network engineers I've experienced can keep a lot in their heads. Maybe it's all the OSPF state machines as well as all the LSAs and what is in them. Or when they are debugging and they run various show commands they can remember which IP was the neighbor that they need to care about. I can't do that. If I have to run a show command that has IP addresses that I need to resolve to know who the neighbors are, I'm in trouble. 

But even if you are good at that, I think it's a very bad habit to rely on that. It appears to me that a lot of networking depends on engineers and operators keeping everything in their head. They aren't in the habit of getting ideas, concepts, etc. out of their heads. This is bad, because when they are in your head, they are unexamined, and as things get more complicated you need to understanding your assumptions and examine them.

**Engineering is the art of tradeoffs.** When making a decision, or even when operating on that decision, you have a set of assumptions about how those things will work. When you don't know what your assumptions are, you can't know that you are making tradeoffs. In real engineering disciplines, those assumptions can turn into numbers, models, and equations. In software,  you get it out in code, and you test it through tests. In networking people are rewarded for keeping all those assumptions in their heads. I want it out. 

I'm much more interested in figuring out how to do Assumption Based Networking than Intent Based Networking. Well, I don't think you can do IBN without recording your assumptions first. If you don't record your assumptions, then you are using somebody else's. And the recording of assumptions is critical.

## Why get assumptions out of your head
* you can't examine and understand tradeoffs for things you don't know that you assume
* you can't have software understand your assumptions and do the right thing unless it's out of your head
* Nobody else knows what your assumptions are.

## So What does this mean?
Let's say I have a core network with 2 routers. The (obvious) assumption is that I can afford for only one of those routers to go down at any one time. When everything is working, I can only afford to have less then 50% capacity of traffic on those routers, because one might fail and I need to be able to support all traffic with a single failure. Sounds good. This means I do things like have capacity alarms at 50%. But mostly, because it's so obvious, there are a bunch of rules I have in my head that I don't even think about. If one of the routers is down, I can't do maintenance on the other. If I am close to 50% utilization, I also need to make sure that all the interfaces on both routers are up all the time. But none of these rules are written down. So as the network changes and as I add automation I need those rules into tools like alarms, and workflows, etc.

Now I'm upgrading to 4 routers. What are my assumptions? Am I assuming I can afford 1 router down, or more? When I go to 8, then what are my assumptions? For each change, each person on the team and each software for management and monitoring needs to know what those rules are. But this is rarely if never done explicitly. And it's complicated, and hard to reason about all the consequences of all the assumptions. 

If I have 8 core routers, I'm probably ok if 2 go down. How do I say for the datacenter, for this layer in the network I can have two routers down before somebody gets woken up, but in this other datacenter, same layer, if 1 goes down, wake somebody up? That's your assumption about your design and the specific instances of your network. You need to change all monitoring systems that produce alarms, any runbooks about when you can make changes or what to troubleshot.



### Examples from the past
One example I had for a long time was that you need to buy big chassis routers for datacenter networks. You need lots of interfaces and they have all the fancy and sophisticated switch backplanes to get you the performance that you need. But why do you need that? It turns out you don't.


## What do we need
There are a lot of different assumptions around networking that we have. I think what we need are a suite of network design tools in which we can record what we think all our requirement assumptions as well as our assumptions about how a design works together. Ok, that's very complicated and not real clear.

For instance, I'd like that you could record the properties of your network design, and then from that make sure that all your monitoring is based on those same assumptions. I think this is the purpose of Intention Based Networking, but I don't think with IBN you an record all your assumptions, I think you have to use the blueprint of what somebody else wrote and they didn't write down all the assumptions.


## What can we do now?
In some ways this is what Intention Based Networking is about, but not really. 

## Suzieq
Try out [Suzieq](https://www.stardustsystems.net/suzieq/), our open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.