---
layout: post
comments: true
author: Justin Pietsch
title: Understanding Networks
excerpt: It's too hard to design, build, scale, and operate networks well.
description: Networking is broken. It's too hard to understand networks, which makes it too hard to design, build, scale, and operate networks the way that we need for today's business and society.
---

IP Networking as an industry, discipline, and practice is broken, badly broken. It's critical to change how this industry works. My best idea about what is broken and what needs to change is that networks are too hard to understand. I think network design is all about understanding. As a network engineer, can I understand what I did 3 months ago, 3 years ago, etc.? Can somebody new understand what I did? Can I understand what will happen as the network changes? Do I understand if the network actually do what is required of it?

I want it to be known that we can and should have a lot better. It's possible, but we need to do a lot of work to get there. We should demand better, and create better, much better.

I want a radically different industry. Obviously, there is a lot that is working in networking, since you know, the Internet and networks rule the world. We need much much better than we have to [Get the network out of the way](https://elegantnetwork.github.io/posts/network-out-of-the-way/). There have been attempts, like SDN, that are much more solutions looking for a problem than they are actually solutions to real problems. "separate control plane from data plane" sure sounds like it means something important, but does it really? (I cannot tell you how many interviews I've done when I asked people what SDN means and gotten that answer. No hire if they can't dive in deeper than that.)

**It's too hard to design, build, scale, and operate networks well.** That is because we don't have the software systems in our field that are necessary, we don't think of what we do as engineering, and we don't look across the whole network systematically. I think the biggest missing piece is good design software and better design practices, but there are major missing pieces for all aspects of network engineering. It's too hard to really understand, to wrap your head around all the decisions that are made about a network, to observe and monitor everything it's doing, and then to turn that information into action and wisdom. How do we deal with the complexity that is networking? How do we make good design decisions that solve the problems for the business that the network supports, as well as then is operatable over time?

There are design and build problems and there are operations problems. I want to separate them. Operational tools also need to get better, but we'll start with design tools. I think everything starts from our lack of good design tools. Better design tools give us better foundations and structures to make better decisions.

Why is design so important? We need to be able to do engineering and study tradeoffs. After we know the network, we need to be able to describe our assumptions and assertion. From this, we can better validate and monitor the network. You can't validate that the network is working correctly if you can't describe how it should be working. Using config validation is wildly insufficient: how do you know that what you configured will do what you expect? If you can write down all your assumptions and expectations, then we can use software to make sure those are true. **How else can you understand your network unless you first understand your own assumptions and expectation and can then verify that those are true?**

I want a system engineering approach to networking. Think of the whole network as a system. How you make a change to one part effects another, so how can you understand those trade offs? I think design and architecture is not really given enough attention and instead operations and tinkering rule the day.

## What's wrong with networking?

* complex and missing design tools
* because of that, don't have the foundational data to base good monitoring on
* focus too much on devices
* can't look at things systematically and holistically
* hard to get all the operational data necessary
* hard to know if your network is healthy of not





Networks are complex and there are lots of interactions. They change over time. Even if you intend for them to stay the same, parts fail, bugs become present, software changes. Almost always there are new requirements for the network. How do we understand the network in that case?
Most of this post is arguing that we need better software, until we get that then we have to keep things very simple, as far as I'm concerned. However, we don't have good design software and very few networks are as simple as possible. So we are trapped in the worst of both worlds; people won't make the network simple and they don't have the software to model and help with understanding.

Of course there are multiple dimensions and networking can get really complex, which makes understanding difficult. I want to just talk about IP connectivity. But of course there's all kinds of virtual networking for containers, Cloud computing, etc. Load balancing, firewalls, etc. There's always new technologies that need to be considered and incorporated. So it's hard to even describe what all the field covers. This makes it hard to be complete and and we won't all agree on where to start.

The protocols aren't the problem. They work fine enough. Or at least, those aren't nearly as big a problem as that we don't have are systematic ways to design, build, scale, and operate networks. I honestly think that the individual router hardware, OS, and software are pretty low on what should be the focus, and they are almost the complete focus. There needs to be changes in the way that we think about networking and how we design, build, scale, and operate networks.

**The devices are the least interesting part of the network, but they get all the attention**. We need to be systematic and understand the whole network as a system, not just one particular piece. It's not that devices aren't interesting, I care about ASICs, and I care about protocol stacks, etc. I fall into this as well, I recently spent a week building benchmarking of different open source BGP stacks. However, these main building blocks are very complicated so the interactions of all the pieces throughout the network and how they are interact is critical to understand, and right now it's too hard to understand how they interact especially as things change, and there is always change.

There's a big problem with our hype cycle. Things get hyped that can't work out and waste a lot of time and energy for operators trying to guess what will work. See SDN below. I don't want to go into it too much, but the software we get doesn't solve the problems we have.

## Why don't we have much better?

This is the most confusing question for me. It could be because we have good enough, and my whole premise is wrong. I know this isn't true for big networks like Amazon, but most people don't have the same problems, so do we have good enough? I don't think so, no matter the network size. Why are we decades behind other fields? Network engineers do not know how to ask for what they need, and software engineers do not know how to operate networks. But there's something else

Network engineers like to tinker, many like to tinker a lot. That kind of tinkering is very useful if it leads to better understanding. But if you do continue to tinker and tinker and not use that to to think systematically that is the problem. I don't think most network engineers even know that they should really be thinking about the whole network, they have no examples and there is no real training.

I think the solution to much of our problems is software, so why aren't network engineers begging for something better? Some part of it is that software for networking comes and goes; it's totally unreliable. What is reliable is that Cisco, Juniper, and Arista will be around, that BGP and OSPF will be around, so that learning how to configure those things will be around. Design software that requires you to think differently is hard work and what real benefit will it be? I don't know how to get around this.

### What about SDN

It depends on what you think SDN means. It sure does seem like SDN has grown to claim everything that uses software. They seem to be claiming that the Cloud providers having their own OSes on their devices is SDN. I almost have no words for how stupid that is since I was a key member of the teams that did that at Amazon. We weren't SDNing.

SDN does describe some real problems, but much of the SDN solutions are a waste of time. Openflow is a disaster, I will try to write a post about that some other day. I hate how much attention really bad ideas get and how much they waste and how far we are because so many of the "future" things are just bad ideas. However, being able to "program the whole network as a distributed system" is a good idea, but you can do that with the current protocols we have if you build a management system that can model the network. I do think one of the big problems in networking is that we can't easily think about the whole system.

The biggest problem with SDN is that it is too much hype; nobody knows what it means. Relatedly, some of the purpose of SDN is to engineer networks without network engineers involved, but that really doesn't work.

Maybe SD-WAN approaches solve this for the regular enterprise WAN needs. What I don't know is how flexible any of these solutions are. How much of it do you really get to make design decisions? I've never used them, but they sound like they are focused on making a system wide decisions on how to make the network be useful. I would like them to be more open so that we can do our own design decision.

I don't know if the SD-WAN style approach toward larger parts of the network, like datacenters or enterprise would work. There are so many different use cases and scenarios and requirements, it seems hard to build a network-in-a-box controller like SD-WAN for more complex networks. Maybe that's the future. I hope it's open and can be understood.



## Systems Engineering

As mentioned above, we focus too much on individual devices. What does the BGP configuration and policy need to be on this device? What we really need to be thinking about is how the whole network will behave. I think that's too hard right now, because we have to way to express system wide policy, we have no way to model what the whole network will do. It's easy to lab up with a single device (or maybe 2) to see if a configuration works. But you are missing how the whole network will work together.

We need a better approach to the network as a system.

## What do we need?

We need better tools and better engineering, they go hand in hand. In my experience better engineering requires better tools. The word tools can get a bad rap because people build specific tools for a specific problem without thinking ahead and that's not what we are talking about here. What other engineering discipline has such terrible tools? Many of them have math and equations, and they all have software to help them design and simulate. Where are our design tools?

I'm demanding that network engineering be an engineering discipline, and not just people who read books from vendors and do what the vendor says. Engineering is about tradeoffs, you must know what tradeoffs you are making and not just following a recipe that somebody else, who isn't a cook (in this analogy I mean a network operator), made for you. There are probably plenty of networks  that going to Cisco and doing what they tell you is enough. But there are a lot of places in which that isn't enough but is still going on. (Using Juniper or Arista or ?? is a little better, at least you are thinking about some tradeoffs, but not much.) Similarly, we need to pursue understanding our networks and understand what will happen as they change.

Our industry focuses too much on per device configuration and not on how the whole system fits together. It's not systematic enough, because there is almost no way to be systematic.

### Do small networks have these problems, or just the public cloud companies

I think so. Different levels of complexity require more support. You can get away with less if you have a less complex network, that's true. On the other hand, when I started at Amazon and we had hundreds of routers having good design tools would have been useful. Again, good design tools can create good data that can be used for great operations.

## Automation isn't enough

Most network engineers think that automation is the way to make networks better. And it is better than not, but it isn't enough. I don't believe automation as is usually practiced makes understanding the network better. It just makes the mechanical processes done by humans done by computers, which can easily make the network harder to understand if not done well, if not done systematically.

We need a much better way of thinking about the network, and much better abstractions to be automated against.

We do need to automate specific changes, but that's very difficult to do well over time without being able to describe how the whole network interacts and our expectations. From that should come specific changes. But until you have describe and modeled the assumptions, intent, etc. of the network then a human has to.

I don't mean to stop automating. I think the way we currently automate is missing a bunch of fundamental information, tools, and infrastructure which makes it hard to do well. It's too ad-hoc. So keep automating if it's all we have, but at the same time with better design tools we can make network automation better, safer, faster, etc.

Some people are using Source of Truth (SOT) systems for their automation, and this is sort of that direction. I think you need to be able to write up the rules for what is in the SOT, and not have to hand manage the data
## What are the consequences

If networking is so broken, what are the consequences? Networks seem to run already. The internet works, pretty well. Cloud companies are gigantic, so what am I talking about? As I tried to describe in [Get the network out of the way](https://elegantnetwork.github.io/posts/network-out-of-the-way/), I think it's too hard to make networks to be what we need them to be, and they don't live up to the needs. We say we are doing our best, which is true, but our current best isn't good enough. Networks are too important.


## Characteristics of tools we need
* get assumptions out of our head
* help us make better design decisions
* populate foundational data to that we can make better decisiona and have better operational tools
* systematic and holistic
* support workflows
* network wide models

## Suzieq

Try out [Suzieq](https://www.stardustsystems.net/suzieq/), the open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.

Suzieq doesn't directly address the problems that I talk about here, but it is about network understanding. And if there was the design tools talked about above Suzieq could integrate better meta-data about the network and be able to give even better insights.

## Conversation / Talk-back

What companies are actually trying to change networking and know what they are doing?

I don't know everything. There might be solutions to some of the problems that I'm describing.cribing. There might be solutions to some of the problems that I'm describing.