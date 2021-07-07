---
layout: post
comments: true
author: Justin Pietsch
title: Understanding Networks
excerpt:
description: Networking is broken. It's too hard to understand networks, which makes it too hard to design, build, scale, and operate networks the way that we need for today's business and society.
---

IP Networking as an industry, discipline, and practice is broken, very badly broken. I think it's critical to change how this industry works. My best idea about what is broken and what needs to change is that networks are too hard to understand. I think network design is all about understanding. As a network engineer, can I understand what I did 3 months ago, 3 years ago, etc.? Can somebody new understand what I did? Can I understand what will happen as the network changes? Do I understand if the network actually do what is required of it?

I want people to know that we can and should have a lot better. It's possible, but we need to do a lot of work to get there.

I want a radically different industry. Obviously, there is a lot that is working in networking, since you know, the Internet and networks rule the world. We need much much better than we have to [get the network out of the way](https://elegantnetwork.github.io/posts/network-out-of-the-way/). There have been attempts, like SDN, that are much more solutions looking for a problem than they are actually solutions to real problems. "separate control plane from data plane" sure sounds like it means something important, but does it really? (I cannot tell you how many interviews I've done when I asked people what SDN means and gotten that answer. No hire if they can't dive in deeper than that.)

**It's too hard to design, build, scale, and operate networks well.** That is because we don't have the software systems in our field that are necessary, we don't think of what we do as engineering, and we don't look across the whole network systematically. I think the biggest missing piece is good design software and better design practices, but there are major missing pieces for all aspects of network engineering. It's too hard to really understand, to wrap your head around all the decisions that are made about a network, to observe and monitor everything it's doing, and then to turn that information into action and wisdom.How do we deal with the complexity that is networking? How do we make good design decisions that solve the problems for the business that the network supports, as well as then is operatable over time? What does operatable mean: can scale appropriately, can make changes to, that I am confident are safe.

I want to begin by focusing on design and the lack of design tools and better processes. There are design and build problems and there are operations problems. I kind of want to separate them. operations tools also need to get better, but we'll start with design tools. I think everything starts from our lack of good design tools. Better design tools give us better foundations and structures to make better decisions.

Why is design so important? We need to be able to do engineering and study tradeoffs. After we know the network, we need to be able to describe our assumptions and assertion. From this, we can better validate and monitor the network. You can't validate that the network is working correctly if you can't describe how it should be working. Using config validation is wildly insufficient: how do you know that what you configured will do what you expect? If you can write down all your assumptions and expectations, then we can use software to make sure those are true. **How else can you understand your network unless you first understand your own assumptions and expectation and can then verify that those are true?**

I want a system engineering approach to networking. Think of the whole network as a system. How you make a change to one part effects another, so how can you understand those trade offs? I think design and architecture is not really given enough attention and instead operations and tinkering rule the day.

## What's wrong with networking?

Networks are complex and there are lots of interactions. They change over time. Even if you intend for them to stay the same, parts fail, bugs become present, software changes. Almost always there are new requirements for the network. How do we understand the network in that case?
Most of this post is arguing that we need better software, until we get that then we have to keep things very simple, as far as I'm concerned. However, we don't have good design software and very few networks are as simple as possible. So we are trapped in the worst of both worlds; people won't make the network simple and they don't have the software to model and help with understanding

Of course there are multiple dimensions and networking can get really complex, which makes understanding difficult. I want to just talk about IP connectivity. But of course there's all kinds of virtual networking for containers, Cloud computing, etc. Load balancing, firewalls, etc. There's always new technologies that need to be considered and incorporated. So it's hard to even describe what all the field covers. This makes it hard to be complete and and we won't all agree on where to start.

Network Engineers and operators are like plumbers, and you wouldn't want plumbers architecting, designing, and building a city water works.
No other engineering discipline has no design tools. Can you image sewers or bridges or cars without software design tools these days? Why don't we have design tools for networks? They are very complicated with potentially lots of pieces, it sure seems like design tools would be useful. Without good software to help us manage the complexity we have to rely on our brains to keep up. I for one cannot keep up with complicated networks. I just don't have the memorization. When we started building our large scale 3-tier Clos in AWS to be our aggregation layer, I fought over and over to make it super simple so that I could model in my head what was going to happen. I spent weeks trying to argue against a practical compromise that was necessary because it was one more complexity that I couldn't keep in my head over time. If we had better software systems to help model these, then it's less necessary.

We are missing software to help design, build, scale, and operate networks. I think this is the biggest problem. Why does networking seem to uniquely be missing the quality of software it needs? It's clearly important enough to merit the software. My best guess is that for software engineers it's much easier to build things that they understand. Software engineers understand word processors and spreadsheets, and they really understand compilers and operating systems. But they have no idea how to actually operate a network. And unless you try really really hard, it's too hard to understand networking to know what software is necessary. At the same time, network engineers are **TERRIBLE** at describing what they need. Actually, I think they don't know what they need, they ask for things that seem to make life more complicated, not less complicated overall. I think more people, especially engineers, are bad at asking for what they need. They usually ask for a specific solution, when they don't understand the other field so they don't understand the tradeoffs being made there. Instead if they **described the problems that they had rather than asking for a better solution** I think we'd make a lot of progress.

The protocols aren't the problem. They work fine enough. Or at least, those aren't nearly as big a problem as that we don't have are systematic ways to design, build, scale, and operate networks. I honestly think that the individual router hardware, OS, and software are pretty low on what should be the focus, and they are almost the complete focus. There needs to be changes in the way that we think about networking and how we design, build, scale, and operate networks.

**The devices are the least interesting part of the network, but they get all the attention**. We need to be systematic and understand the whole network as a system, not just one particular piece. It's not that devices aren't interesting, I care about ASICs, and I care about protocol stacks, etc. I fall into this as well, I recently spent a week building benchmarking of different open source BGP stacks. However, these main building blocks are very complicated so the interactions of all the pieces throughout the network and how they are interact is critical to understand, and right now it's too hard to understand how they interact especially as things change, and there is always change.

There's a big problem with our hype cycle. Things get hyped that can't work out and waste a lot of time and energy for operators trying to guess what will work. See SDN below. I don't want to go into it too much, but the software we get doesn't solve the problems we have.

## Why don't we have much better?

This is the most confusing question for me. It could be because we have good enough, and my whole premise is wrong. I know this isn't true for big networks like Amazon, but most people don't have the same problems, so do we have good enough? I don't think so, no matter the network size. Why are we decades behind other fields? Network engineers do not know how to ask for what they need, and software engineers do not know how to operate networks. But there's something else

Network engineers like to tinker, many like to tinker a lot. That kind of tinkering is very useful if it leads to better understanding. But if you do continue to tinker and tinker and not use that to to think systematically that is the problem. I don't think most network engineers even know that they should really be thinking about the whole network, they have no examples and there is no real training.

I think the solution to much of our problems is software, so why aren't network engineers begging for something better? Some part of it is that software for networking comes and goes; it's totally unreliable. What is reliable is that Cisco, Juniper, and Arista will be around, that BGP and OSPF will be around, so that learning how to configure those things will be around. Design software that requires you to think differently is hard work and what real benefit will it be? I don't know how to get around this.

### What about SDN

First off, it depends on what you think SDN means. It sure does seem like SDN has grown to claim everything that uses software. They seem to be claiming that the Cloud providers having their own OSes on their devices is SDN. I almost have no words for how stupid that is since I was a key member of the teams that did that at Amazon. We weren't SDNing. 

SDN does describe some real problems, but much of the SDN solutions are a waste of time. Openflow is a disaster, I will try to write a post about that some other day. I hate how much attention really bad ideas get and how much they waste and how far we are because so many of the "future" things are just bad ideas. However, being able to "program the whole network as a distributed system" is a good idea, but you can do that with the current protocols we have if you build a management system that can model the network. I do think one of the big problems in networking is that we can't easily think about the whole system.

The biggest problem with SDN is that it is too much hype, so nobody knows what it means. Relatedly, some of the purpose of SDN is to engineer networks without network engineers involved, but that really doesn't work.

Maybe SD-WAN approaches solve this for the regular enterprise WAN needs. What I don't know is how flexible any of these solutions are. How much of it do you really get to make design decisions? I've never used them, but they sound like they are focused on making a system wide decisions on how to make the network be useful. I would like them to be more open so that we can do our own design decision.

I don't know if the SD-WAN style approach toward larger parts of the network, like datacenters or enterprise would work. There are so many different use cases and scenarios and requirements, it seems hard to build a network-in-a-box controller like SD-WAN for more complex networks. Maybe that's the future. I hope it's open and can be understood.

## What do we need?

We need better tools and better engineering, they go hand in hand. In my experience better engineering requires better tools. The word tools can get a bad rap because people build specific tools for a specific problem without thinking ahead and that's not what we are talking about here. What other engineering discipline has such terrible tools? Many of them have math and equations, and they all have software to help them design and simulate. Where are our design tools?


I'm demanding that network engineering be an engineering discipline, and not just people who read books from vendors and do what the vendor says. Engineering is about tradeoffs, you must know what tradeoffs you are making and not just following a recipe that somebody else, who isn't a cook (in this analogy I mean a network operator), made for you. There are probably plenty of networks  that going to Cisco and doing what they tell you is enough. But there are a lot of places in which that isn't enough but is still going on. (Using Juniper or Arista or ?? is a little better, at least you are thinking about some tradeoffs, but not much.) Similarly, we need to pursue understanding our networks and understand what will happen as they change.


Our industry focuses too much on per device configuration and not on how the whole system fits together. It's not systematic enough, because there is almost no way to be systematic.

## Examples of network engineering

Engineering is the art of making tradeoffs which means I want to be able to ask a bunch of questions. How can I do that now? I don't know anything about real engineering disciplines, but I'd bet that they do a lot of what-if scenarios and simulations. In networking the best we can do is great engineers on a whiteboard arguing about what will happen. This is very useful, and it works, but it's insufficient with how important networks. There's no technical reason that we can't make our decision with much better data, including numbers some times.

Here are some of the kinds of questions I'd like to be able to ask:

### Design time

I'd like to try out different routing protocols in my three tier clos. Facebook and Microsoft say that BGP is better than OSPF, [I beg to differ](https://elegantnetwork.github.io/posts/What-Ive-learned-about-OSPF/), but wouldn't you like to be able to actually try out the differences for yourself? Wouldn't it be nice to be able to model up a large network with either BGP or OSPF (or both), and then be able to simulate that? There are simulation systems, like GNS3, but it's generally too hard to build all the configuration to do that. How will convergence be affected by these choices? Not just generally, but with numbers, actually measured, so that you can compare.

I can't understand how my network will handle failure. I need to understand how device, interface, and even protocol failures could cause me problem. At design time it would be nice to see what happens during failure.

These are the kind of complex routing policy changes that I need to make. Are they likely to be safe? Can this network in general handle these type of changes?

### Ongoing operations

I said I'm not going to talk about operational issues, but I'm going to slip up and talk about some of the questions we need to answer. I think you can't answer these well unless you have the better design tools.

Do I need to care about this current failure? While things are running, there are often (almost always) some kind of failures. For any given failure, at any particular time, is this is important? If a device fails does somebody need to get woken up? This is actually hard to understand unless you can really model the network. Also, you have have to encode the rules that the engineers were assuming when the network was designed. In this layer of the network with four devices, is it okay if two devices fail, or just one?

Is it safe to make this current change? On a clean network this would work, but is the current running network in the state necessary to be running fine if I make this change? Do I have enough capacity to do maintenance? What else could go wrong right now if I made this change?

I need to have a complex routing policy and I need to know that it's going to work the way I think it will work. It sure would be helpful if I could describe how I think it's going to work.

## Systems Engineering

As mentioned above, we focus too much on individual devices. What does the BGP configuration and policy need to be on this device? What we really need to be thinking about is how the whole network will behave. I think that's too hard right now, because we have to way to express system wide policy, we have no way to model what the whole network will do. It's easy to lab up with a single device (or maybe 2) to see if a configuration works. But you are missing how the whole network will work together.

We need a better approach to the network as a system.


## Automation isn't enough

I think for many, they think that automation is the way to make networks better. I don't believe automation as is usually practiced makes understanding the network better. It just makes the mechanical processes done by humans done by computers, which can easily make the network harder to understand if not done well, if not done systematically.

We need a much better way of thinking about the network, and much better abstractions to be automated against.

THe problem isn't that we need to automate specific changes, it's that we need to be able to describe how the whole network interacts and our expectations. From that should come specific changes. But until you have describe and modeled the assumptions, intent, etc. of the network then a human has to

## A more concrete example

**maybe this is another post**

I've been pretty abstract so let me describe where I'm coming from. In 2009, we started trying to figure out how to use commodity ASICs in pizza boxes for our aggregation network. The devices we had were based on Broadcom Scorpion, a 10G x 48 device. With a three tier Clos, that's 2880 devices. I knew that in the future we'd have 64 port devices, which is a max of 5120. The Network OS that we had only had OSPF. Do you know how to get OSPF to work across 5K devices? Can it be done? How? I certainly didn't know how. As we worked it out on a whiteboard, I needed to see 


From that became our configuration management systems. 
## What are the consequences

If this is so broken, what are the consequences? Networks seem to run already. THe internet works. Cloud companies are gigantic, so what am I talking about? As I tried to describe in [Get the network out of the way](https://elegantnetwork.github.io/posts/network-out-of-the-way/), I think it's too hard to make networks to be what we need them to be, and they don't live up to the needs. We say we are doing our best, which is true, but our current best isn't good enough. Networks are too important.


## What is needed for all networks, large or small

### What do extremely large networks need


## What do I want built?

As mentioned, I think you have to start with the foundational information, which is design tools. From that you need to capture all the important data about a network.

### Design tools

A way of describing high level design intent, assumptions, decisions, and from that be able to create device configuration for any device.


We need the centralized control that SDN promised, but not necessarily in a centralized controller. You want to configure and manage everything together centrally, but you can still have the same distributed routing system.

routing protocol policy in specific needs a system wide

I want the ability to make my own reference design. I want to be able to understand the tradeoffs in that design. ANd then I want to be able to create instances of that design, and make changes to it over time.

when making changes
 *

##  Solutions

There might be some good answers some of the networking companies, I haven't had a chance to try them all out.

what companies are really changing the way that networking is done? Not in the OpenFlow like way, which would never work, nor in the I don't actually understand the problem but I'm going to do something hypey. Probably not SDN. Controllers are a good idea for some problems, but most companies that talk about SDN are not solving real problems.

I think [Apstra](https://apstra.com/) might be working on this, but I don't know in detail how they work. I've never actually used Apstra, and the only time I talked to them was when I was at Amazon, and they couldn't scale and at least at the time didn't allow us to really get into the details of describing a design ourselves. I'm afraid that Apstra is too cookie cutter and doesn't really allow me to be able to specify the architecture that I want at a high level.

I'm also curious about [Gluware](https://gluware.com/), [Itential](https://www.itential.com/), [Anuta Networks](https://www.anutanetworks.com/). At some point I need to dive into [Cisco NSO] (https://www.cisco.com/c/en/us/products/cloud-systems-management/network-services-orchestrator/index.html), but I don't trust Cisco. I know that they bought the company, but still, it's so very hard to trust Cisco software. And I've never tried out [Arista's CloudVIsion](https://www.anutanetworks.com/), if it's not multivendor I think it's useless. Both because I can't imagine wanting to be locked into a single vendor, but also because if you don't have to do mutlivendor then you don't do a good job of really modeling the network.

I'm suspicious of all of these, I doubt that they are what we need, but I haven't tried them enough. Usually things like this focus way too much on visual and drag-n-drop. That's not the real problem. The real problems are that I can't even describe my design expectations and assumptions and that I can't model based on those things.








## Conversation / Talk-back

What companies are actually trying to change networking and know what they are doing?


I don't know everything. There might be solutions to some of the problems that I'm describing.


## Not sure what to do with this

At Amazon when we were first designing the very large 3-tier Clos that makes up the aggregation network in the Amazon/AWS datacenters, some people were saying that recabling as we grow isn't that big a deal, and that datacenter techs aren't necessarily that expensive. True, however, recabling is a very tedious and potentially error prone process that is invasive and potentially very damaging. I tried to design a network that didn't require recabling. I didn't completely succeed because there were some scaling issues I couldn't deal with: I would have had to overbuild in many cases to avoid recabling in a couple cases.

However, Amazon/AWS has also built a sophisticated system for checking when cabling is done. A new network is thousands of cables and optics, making sure they are all cabled correctly and also that all the optics and cables work well is super tedius and so takes software to make it better. Again, when we made the trade off to build large Clos networks from Single ASIC pizza boxes rather than large chassis, the tradeoff is that you have a lot more cables, potentialy an order of magnitude more, so you have to do something different and hopefully use software to make things better.