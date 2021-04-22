---
layout: post
comments: true
author: Justin Pietsch
title: Openflow: a dangerously bad idea
excerpt: 
description: 
---

I recently saw a blog post [celebrating Openflow](https://opennetworking.org/news-and-events/blog/openflow-catalyst-that-kickstarted-the-sdn-transformation/) and it's contribution to networking. I'm not sure talking about Openflow at this point is worth anybody's time, but for some reason, I'm going to. 

Openflow is a bad idea, it was always a bad idea, and it caused (and still causes) a big problem in our industry. Openflow does not solve a real problem. The abstraction is wrong and makes managing the networks and operating them worse than before it. That wouldn't be so bad, but it's also taken attention away from solving real problems, and that's why it's dangerous.

Yes, I do know that Google used Openflow in their Orion SDN controller. However, I continue to think there are other ways they could have achieved the same thing. 

This is a post that's hard to write well. This topic makes me angry and so it's had to write something that everyone can read, but there are people I want to read this who can be offended by my opinion or just write me off because I disagree and am emotional about it. I want a real discussion because I think there are some really important points for our industry. It doesn't really matter that I don't like Openflow, it matters why I don't like Openflow

## Why listen to me?

It's easy for researchers or people trying to disrupt networking to say that network engineers just don't like new ideas and are stuck in the past. This can be true, but we are talking about a specific engineer with a specific opinion. 

I'm worked in the largest datacenter network in the world: AWS, for a very long time. I'm very familiar with understanding tradeoffs around where functionality should be in the network (or on the hosts). I was around when EC2 was originally created and part of the networking team that ensured that the virtual networking for EC2 was not in the network, but was on the hosts. When VPCs were created, again, we made sure that the virtual network was not in the physical network. 

So the problem for me with Openflow is not that I want networking in the network. It's that it's a bad idea. It's the wrong abstraction and it doesn't solve the real problems I have especially in my datacenters.

## Why is Openflow a bad idea?

Openflow points out some important problems, but solves them in a terrible way.  Openflow was made so that researchers could more easily write new ways of directing traffic. That's not a good reason to change production networking. They came up with the idea of a centralized controller. Centralized controllers have an important role in some parts of the network, but not all. 

In almost every network, **flows are the wrong abstractions to make decisions on**. They are the right abstractions for load balancers and firewalls, which are generally on the edge of the network, but for routing inside of a network, flows are the wrong abstraction.

We've known this as an industry. In the 90s, Cisco had routers and switches that were flow based. The original use of Netflow was to make forwarding decisions. However, the industry quickly learned this is a very bad idea. There will always be many orders of magnitude more flows than there are routes in the routing table. Also, a very important piece of scaling IPv4 routing is route aggregation. How do you aggregate flows? Can you aggregate flows? Well, I mean you can aggregate them into IP addresses, but  why didn't you start that way in the first place?

In the early 2000s a company called Caspian Networks tried to sell routers that could optimize traffic based on flow data. I never understood that company, it was clearly a bad idea. I don't know anybody who bought their products, and they fizzled out.

For Openflow to be flow based, and then worse off to have the first packet of every flow go to a centralized controller is a terrible idea. This was known in the industry.

When EC2 launched, well, maybe I shouldn't tell that story in public. It involves flow routing and first packets going to an under powered separate processor and a lot of pain and suffering. Daily meeting with not-enough-progress, sad, very sad customers, etc. Oh yeah, and the very first ... yeah, not that one either

I hate when bad ideas come back and are trumpeted by people  as some brilliant idea. It wasn't a good idea the first time, it isn't a good idea now.

We wasted so much time and attention on an idea that **could never work**. It cannot scale. It adds a lot of complexity. And let me say it again, the first packet went to a separate controller.  That should never have shipped.

I don’t want the control that it seems to imply. If you remember early Openflow demos, or especially it’s predecessor Ethane, they were about directing flows specifically  through a network.  I don’t ever want that. I just want connectivity everywhere. In a datacenter that's what you want.

In WAN you do want more control, but Openflow isn't a good abstraction for that. Traffic engineering is better with a controller, because it is possible to make better decisions with a centralized view, so that part of SDN right, but not first packet goes to the controller. That's still terrible.

I'm all for shipping things that are useful but not completely done. Iterative software and all that. But first packet goes to a controller is such a bad idea it should never have been shipped or propagated as a good idea.

I  also get upset when I read that the problem with Openflow is that the hardware was wrong. It’s not that the hardware was wrong. The abstraction was never going to work. 


## Why is Openflow dangerous?

As an industry wasted so much time and attention on this bad idea that could never be useful. There are so many other things that we didn’t get right. 

## What about SDN?

Some things are better done centrally. Configurations do an management. Traffic engineering.  But lots of things are not, like per flow setup. Holy cow what a bad bad bad idea

There's "Separate the control plane from the forwarding plane" promise of SDN. Why is it important to separate controls plane from forwarding plane? It’s not. The point is to be able to have control over the decision making. BUt there are other and better ways to do that. It is possible to have an open source router in which you have source for all the software. In this case why do I need to separate control plane from routing plane. It sounds cool, I'll give you that.

## What should we be working on if it isn't Openflow

I care about management and monitoring and getting assumptions out of our heads.  I care that there is no good way to express my BGP policy across the network, I have to figure out step by step across the network how I want it to perform. That's crazy, super error prone, and terrible for our networks.

I care that monitoring devices is still too hard. Network Vendors don't prioritize being able to get access to all data programmatically and being sure that all errors (including all packet drops) are recording and counts

I care that management of devices is too hard. There aren't good ways of expressing how I want my whole network to work; it's all a device at a time. And again the network vendors don't prioritize management, the APIs are inconsistent and buggy.

I care about really well tested networks and devices.

## The problem of research

Many researchers will discount my argument because I don't understand research. That's true. I think most research is a waste of time. Or at least can never be practical. I sure wish researchers would be brave enough to work on an idea and come to the end and say, this was interesting, but not a good idea to actually use. That would be awesome. Instead we get bad ideas, that the next PhD candidate doesn't understand is a bad idea and then they are built on.

But I think there's some really interesting research. Batfish and Forward Networks came out of research and that's really important. Formal verification of things like ACLs is super interesting and I wish was widely deployed.

I think propane is really important and am hoping that somebody (hopeful me) figures out how to take similar ideas and turn it into a real product/project. It has the potential to solve very important problems. There is important research, but translating from researching to product is very hard, and getting research to understand how to actually operate a network is a very high mountain to climb.

## Conclusion

I don't understand why the shiny things that people go after in networking are often so terrible. Openflow is the ultimate example of shiny, useless and dangerous. 

Let's build better abstractions, solve real problems.