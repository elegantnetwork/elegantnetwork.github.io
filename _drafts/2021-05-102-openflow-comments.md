---
layout: post
comments: true
author: Justin Pietsch
title: "Openflow: a dangerously bad idea"
excerpt:
description:
---

I recently saw a blog post [celebrating Openflow](https://opennetworking.org/news-and-events/blog/openflow-catalyst-that-kickstarted-the-sdn-transformation/) and it's contribution to networking. I'm not sure talking about Openflow at this point is worth anybody's time, but it brings up rancor in me.

**Openflow is a bad idea, it was always a bad idea**, and it caused (and still causes) a big problem in our industry. Openflow does not solve a real problem that operators have in their networks. The abstraction of a flow is the wrong abstraction for thinking about forwarding decisions and makes managing the networks and operating them worse than before it. That wouldn't be so bad if it hadn't taken attention away from solving real problems, and that's why it's dangerous.

This is a post that's hard to write well. This topic makes me angry and so it's had to write something that everyone can read, but there are people I want to read this who can be offended by my opinion or just write me off because I disagree and am emotional about it. I want a real discussion because I think there are some really important points for our industry. It doesn't really matter that I don't like Openflow, **it matters why I don't like Openflow.**

I wish there was a way to rebuttal bad ideas. This is what I'm trying to do here.

## Why is Openflow a bad idea?

Openflow points out some important problems, but solves them in a terrible way Openflow was made so that researchers could more easily write new ways of directing traffic. That's not a good reason to change production networking. They came up with the idea of a centralized controller. Centralized controllers have an important role in some parts of the network, but not all. It's much easier to make traffic engineering decisions well in a centralized system. But Openflow was supposed to solve all kinds of problems which it doesn't solve.

One of the purposes of Openflow is to have a standard API between control plane and forwarding plane. Why? I still don't know why I need that. There are already standard enough ways to program routes, that is usually enough to get done what is necessary.

In almost every network, **flows are the wrong abstractions to make decisions on**. They are the right abstractions for load balancers and firewalls, which are generally on the edge of the network, but for routing inside of a network, flows are the wrong abstraction. You cannot scale flow based forwarding decisions.

We've known this as an industry. In the late 90s, Cisco had routers and switches that were flow based. The original use of Netflow was to make forwarding decisions. However, the industry quickly learned this is a very bad idea because you will run out of hardware resources and then your performance will crash. There will always be several orders of magnitude more flows than there are routes in the routing table. Also, a very important piece of scaling IPv4 routing is route aggregation. How do you aggregate flows? Can you aggregate flows? Well, I mean you can aggregate them into IP addresses, but why didn't you start that way in the first place?

In the mid 2000s a company called Caspian Networks tried to sell routers that could optimize traffic based on flow data. I never understood that company, it was clearly a bad idea. I don't know anybody who bought their products, and they fizzled out. They promised specialized priority for specific flows, but again, how important is that? It's not worth the trade-offs.

For Openflow to be flow based, and then worse off to have the first packet of every flow go to a centralized controller is a terrible idea. This was known in the industry.

Very early in EC2 some poor decisions were made and we had to deal with some terrible flow routing that we didn't understand, including that the first packet of a flow had to go to a very slow processor. It was terrible, lots of suffering. Daily meeting with not-enough-progress, sad, very sad customers, etc. The idea in this post that flows are a bad idea is not abstract; flow based decision making through a network is non-scalable, I've felt the pain and the industry has already felt the pain before.

**I hate when bad ideas come back and are trumpeted by people as some brilliant idea.** It wasn't a good idea the first time, it isn't a good idea now.

We wasted so much time and attention on an idea that **could never work**. It cannot scale. It adds a lot of complexity. And let me say it again, the first packet went to a separate controller. That should never have shipped.

I don’t want the control that it seems to imply. If you remember early Openflow demos, or especially it’s predecessor Ethane, they were about directing flows specifically through a network. I don’t ever want that; it's not worth the extra complexity and scale problems. I just want connectivity everywhere. In a datacenter that's what you want.

In WAN you do want more control, but Openflow isn't a good abstraction for that. Traffic engineering is better with a controller, because it is possible to make better decisions with a centralized view. That part of SDN right, but not first packet goes to the controller. That's still terrible.

I'm all for shipping things that are useful but not completely done. Iterative software and all that. But first packet goes to a controller is such a bad idea it should never have been shipped or propagated as a good idea. I also get upset when I read that the problem with Openflow is that the hardware was wrong. It’s not that the hardware was wrong. The abstraction was never going to work and isn't what we want.

## What about SDN?

Some things are better done centrally. Configuration and management is better done centrally. Traffic engineering. But lots of things are not, like per flow setup. Holy cow what a bad bad bad idea

There's "Separate the control plane from the forwarding plane" promise of SDN. Why is it important to separate controls plane from forwarding plane? It’s not. The point is to be able to have control over the decision making. But there are other and better ways to do that. It is possible to have an open source router in which you have source for all the software. In this case why do I need to separate control plane from routing plane. It sounds cool, I'll give you that.
## Why listen to me?

It's easy for researchers or people trying to disrupt networking to say that network engineers just don't like new ideas and are stuck in the past. This can be true, but we are talking about a specific engineer with a specific opinion.

I'm worked in the largest datacenter network in the world: AWS, for a very long time. I'm very familiar with understanding tradeoffs around where functionality should be in the network (or on the hosts). I was around when EC2 was originally created and part of the networking team that ensured that the virtual networking for EC2 was not in the network, but was on the hosts. When VPCs were created, again, we made sure that the virtual network was not in the physical network.

So the problem for me with Openflow is not that I want networking in the network. It's that it's Openflow in specific is a terrible idea. It's the wrong abstraction and it doesn't solve the real problems I have especially in my datacenters.

## Why not listen to me?

Google had an SDN controller for a while: Orion. Yes, I do know that Google used Openflow in their Orion SDN controller. However, I continue to think there are other ways they could have achieved the same thing that would have been much better ways of doing the same thing. I've heard that they've move away from Openflow.

Larry Peterson is a lot smarter and more famous than me. He writes good books.

## What should we be working on if it isn't Openflow

I care about management and monitoring and getting assumptions out of our heads. I care that it's too hard to understand networks. I care that there is no good way to express my BGP policy across the network, I have to figure out step by step across the network how I want it to perform. That's crazy, super error prone, and terrible for our networks.

I care that monitoring devices is still too hard. Network vendors don't prioritize being able to get access to all data programmatically and being sure that all errors (including all packet drops) are recording and counts

I care that management of devices is too hard. There aren't good ways of expressing how I want my whole network to work; it's all a device at a time. And again the network vendors don't prioritize management, the APIs are inconsistent and buggy.

I care about really well tested networks and devices.

## The problem of research

Researchers might discount my argument because I don't understand research. That's true. I think most research is a waste of time. Or at least can never be practical. I sure wish researchers would be brave enough to work on an idea and come to the end and say, this was interesting, but not a good idea to actually use. That reporting negative results counted as much as supposedly positive results That would be awesome. Instead we get bad ideas, that the next PhD candidate doesn't understand is a bad idea and then they are built on.

But I think there's some really interesting research. Batfish and Forward Networks came out of research and that's really important. Formal verification of things like ACLs is super interesting and I wish was widely deployed.

I think languages to help describe policy like [propane](https://propane-lang.org/) are really important and am hoping that somebody (hopeful me) figures out how to take similar ideas and turn it into a useable product/project. It has the potential to solve very important problems. There is important research, but translating from researching to product is very hard, and getting research to understand how to actually operate a network is a very high mountain to climb.

## Conclusion

I don't understand why the shiny things that people go after in networking are often so terrible. Openflow is the ultimate example of shiny, useless and dangerous.

Let's build better abstractions, solve real problems.

## Suzieq

Try out [Suzieq](https://www.stardustsystems.net/suzieq/), the open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network. 