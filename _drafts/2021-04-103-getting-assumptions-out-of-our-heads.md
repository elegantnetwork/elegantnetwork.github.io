---
layout: post
comments: true
author: Justin Pietsch
title: Getting Assumptions out of our Heads
excerpt:
description: Too much of networking requires network engineers to keep a lot in their heads. Getting those thoughts out, especially in a programmable system, means that you can then truly understand what you are doing.
---
Networking requires a lot of decisions by many different people. Every point of designing, building, scaling, and operating networks requires decisions. Many of those decisions have even more assumptions behind them. In networking we don't have a standard and good way of saving those artifacts; they live in people's heads and because of that we can't act on them, make them better, and use software to enforce and verify our assumptions.

Some of the best network engineers I've experienced can keep a lot in their heads. Maybe it's the OSPF state machines as well as all the LSAs and what is in them. Or when they are debugging and they run various show commands they can remember which IP was the neighbor that they need to care about. I can't do that. If I have to run a show command that has IP addresses that I need to resolve to know who the neighbors are, I'm in trouble. At some point that happens to everybody. It would be a better idea of we didn't need that.

Even if you are good at that, I think it's a very bad habit to rely on that. It appears to me that a lot of networking depends on engineers and operators keeping everything in their head. They aren't in the habit of getting ideas, concepts, etc. out of their heads. This is bad, because when they are in your head, they are unexamined, and as things get more complicated you need to understanding your assumptions and examine them.

**Engineering is the art of tradeoffs.** When making a decision, or even when operating on that decision, you have a set of assumptions about how those things will work. When you don't know what your assumptions are, you can't know that you are making tradeoffs. In real engineering disciplines, those assumptions can turn into numbers, models, and equations. In software "engineering", you get it out in code, and you test it through (hopefully automated) tests. In networking people are rewarded for keeping all those assumptions in their heads. I want them out. I want them out of our heads **RIGHT NOW**.

I'm much more interested in figuring out how to do Assumption Based Networking than Intent Based Networking. Well, I don't think you can do IBN without recording your assumptions first. If you don't record your assumptions, then you are using somebody else's. The recording of assumptions is critical. To say it another way, when I want my intent fullfilled, I have assumptions about how that will happen. If I can't examine, understand, or change how that is being done I often can't build what I need to build.

### Why get assumptions out of your head?

* You can't examine and understand tradeoffs for things you don't know that you assume.
* You can't have software understand your assumptions and do the right thing unless it's out of your head.
* Nobody else knows what your assumptions are.

## What does this mean?

Let's say I have a core network with 2 routers. The (obvious) assumption is that I can afford for only one of those routers to go down at any one time. When everything is working, I can only afford to have less then 50% capacity of traffic on those routers, because one might fail and I need to be able to support all traffic with a single device. Sounds good. This means I do things like have capacity alarms at 50%. But mostly, because it's so obvious, there are a bunch of rules I have in my head that I don't even think about. If one of the routers is down, I can't do maintenance on the other. If I am close to 50% utilization, I also need to make sure that all the interfaces on both routers are up all the time. But none of these rules are written down. So as the network changes and as I add automation I need those rules into tools like alarms, and workflows, etc.

Now I'm upgrading to 4 routers. What are my assumptions? Am I assuming I can afford 1 router down, or is it still 1/2? When I go to 8, then what are my assumptions? For each change, each person on the team and each software for management and monitoring needs to know what those rules are. But this is rarely if ever done explicitly. It's complicated, and hard to reason about all the consequences of all the assumptions.

If I have 8 core routers, I'm probably ok if 2 go down. How do I say for the datacenter, for this layer in the network I can have two routers down before somebody gets woken up, but in this other datacenter, same layer, if 1 goes down, wake somebody up? That's your assumption about your design and the specific instances of your network. You need to change all monitoring systems that produce alarms, any runbooks about when you can make changes or what to troubleshot.

In more mature network engineering organizations, design documents are written and hopefully many of the assumptions can be written and reviewed at that time. This is really important and a great practice that everybody should do. However, if they are not in a system that software can also use them to verify and enforce, then you are missing out a huge area of conflict and errors.

We need ways to get assumptions out of our heads, into systems that humans and software can interrogate.

### Other examples of assumptions

* Your ops team is getting too many tickets and you assume that 100G interfaces are more important than 10G interfaces so you turn off paging for 10G interfaces, but somebody else doesn't know that and sets up new monitoring, sees the non-paging on 10G interfaces and assumes that's for everything.
* You assume a device can be down for RMA for 4 hours (or 8 or 24 or 3 days). But the operations manager doesn't know that and so doesn't prioritize devices down until later.
* You assume your monitoring systems will tell you a device is down right away. But the monitoring system polls every 5 minutes and requires 3 SNMP failures to make something down, so you are surprised when it takes 15 minutes to tell you a device is down.

## What do we need?

There are a lot of different assumptions around networking that we have. I think what we need are a suite of network design tools in which we can record what we think all our requirement assumptions as well as our assumptions about how a design works together. That seems there is a huge gap between what we have (effectively nothing) and where we need to be. Yes, that is true. But we should still think about what we need.

For instance, I'd like that you could record the properties of your network design, and then from that make sure that all your monitoring is based on those same assumptions. I think this is the purpose of Intention Based Networking, but I don't think with IBN you an record all your assumptions, I think you have to use the blueprint of what somebody else wrote and they didn't write down all the assumptions.

## What can we do now?

In some ways this is what Intention Based Networking is about, but not really. I think they have similar goals, but capturing assumptions is a critical missing step. I've not used any product labeled IBN, but to the best of my understanding you declare things about the network, like how many client interfaces you want. But the assumptions about how those intentions are fulfilled are not yours to record. You must use what was designed for you in their blueprints. I'm sure there is some amount of configurability in the blueprints, but I doubt you get to record your assumptions about how your are changing the blueprints.

Probably the best place to start is in writing out assumptions in design docs. Even if you are the only network engineer it's a good idea to start explicitly thinking about what your assumptions are. The more that your automation uses algorithms and loops to do things rather than standard config means you are recording assumptions, though not in the most convenient way. For instance, it's much better to have a loop in code decide your IP addresses for interfaces than a static table.

After that, the more assumptions that can get into code the better, though it also requires that you document why you are making the assumptions you are making.

## Summary

It's important to understand your assumptions around designing, building, managing, and operating your network. With that you can examine them and use them as the basis for all your automation, management, and monitoring.
## Suzieq

Try out [Suzieq](https://www.stardustsystems.net/suzieq/), the open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.

Suzieq has asserts in which you can have written some of the things that you think must be true about your network. The current asserts are pretty powerful, but are a long way off from where we know we will take them. Try out Suzieq and help us make it a great platform for this problem and others.
.
