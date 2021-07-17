---
layout: post
comments: true
author: Justin Pietsch
title: Network Design Tools
excerpt: It's too hard to design, build, scale, and operate networks well.
description: We need real networking design tools.
---

IP Networking as an industry, discipline, and practice is broken, badly broken. It's critical to change how this industry works. My best idea about what is broken and what needs to change is that networks are too hard to understand. I think network design is all about understanding. As a network engineer, can I understand what I did 3 months ago, 3 years ago, etc.? Can somebody new understand what I did? Can I understand what will happen as the network changes? Do I understand if the network actually do what is required of it?

Obviously, there is a lot that is working in networking, since you know, the Internet and networks rule the world. We need much much better than we have to [Get the network out of the way](https://elegantnetwork.github.io/posts/network-out-of-the-way/). There have been attempts, like SDN, that are much more solutions looking for a problem than they are actually solutions to real problems. "separate control plane from data plane" sure sounds like it means something important, but does it really? (I cannot tell you how many interviews I've donewhen I asked people what SDN means and gotten that answer. No hire if they can't dive in deeper than that.)

**It's too hard to design, build, scale, and operate networks well.** That is because we don't have the software systems in our field that are necessary, we don't think of what we do as engineering, and we don't look across the whole network systematically. I think the biggest missing piece is good design software and better design practices, but there are major missing pieces for all aspects of network engineering. It's too hard to really understand, to wrap your head around all the decisions that are made about a network, to observe and monitor everything it's doing, and then to turn that information into action and wisdom. How do we deal with the complexity that is networking? How do we make good design decisions that solve the problems for the business that the network supports, as well as then is operatable over time?

There are design and build problems and there are operations problems. I want to separate them. Operational tools also need to get better, but we'll start with design tools. I think everything starts from our lack of good design tools. Better design tools give us better foundations and structures to make better decisions.

Why is design so important? We need to be able to do engineering and study tradeoffs. After we know the network, we need to be able to describe our assumptions and assertion. From this, we can better validate and monitor the network. You can't validate that the network is working correctly if you can't describe how it should be working. Using config validation is wildly insufficient: how do you know that what you configured will do what you expect? If you can write down all your assumptions and expectations, then we can use software to make sure those are true. **How else can you understand your network unless you first understand your own assumptions and expectation and can then verify that those are true?**

## Where did this idea come from?

I've been pretty abstract so let me describe where I'm coming from. In 2009, we started trying to figure out how to use commodity ASICs in pizza boxes for our aggregation network. The devices we had were based on Broadcom Scorpion, a 10G x 48 device. With a three tier Clos, that's 2880 devices. I knew that in the future we'd have 64 port devices, which is a max of 5120. The Network OS that we had only had OSPF. Do you know how to get OSPF to work across 5K devices? Can it be done? How? I certainly didn't know how. As we worked it out on a whiteboard, I needed to see it running. What was available wasy Dynamips with Dynagen, which would run Cisco IOS device images in a virtual machine. I wrote software to build out these Clos Networks and see simple things like convergence and


centralized view of configuration. it would be better to use intent, declarative, and be able to get all the assumptions out of our head.

## Is Networking an Engineering discipline?

Networks are complex and there are lots of interactions. They change over time. Even if you intend for them to stay the same, parts fail, bugs become present, software changes. Almost always there are new requirements for the network. How do we understand the network in that case?
Most of this post is arguing that we need better software, until we get that then we have to keep things very simple, as far as I'm concerned. However, we don't have good design software and very few networks are as simple as possible. So we are trapped in the worst of both worlds; people won't make the network simple and they don't have the software to model and help with understanding.

Network Engineers and operators are like plumbers, and you wouldn't want plumbers architecting, designing, and building a city water works.
No other engineering discipline has no design tools. Can you image sewers or bridges or cars without software design tools these days? Why don't we have design tools for networks? They are very complicated with potentially lots of pieces, it sure seems like design tools would be useful. Without good software to help us manage the complexity we have to rely on our brains to keep up. I for one cannot keep up with complicated networks. I just don't have the memorization. When we started building our large scale 3-tier Clos in AWS to be our aggregation layer, I fought over and over to make it super simple so that I could model in my head what was going to happen. I spent weeks trying to argue against a practical compromise that was necessary because it was one more complexity that I couldn't keep in my head over time. If we had better software systems to help model these, then it's less necessary.

We are missing software to help design, build, scale, and operate networks. Why does networking seem to uniquely be missing the quality of software it needs? It's clearly important enough to merit the software. My best guess is that for software engineers it's much easier to build things that they understand. Software engineers understand word processors and spreadsheets, and they really understand compilers and operating systems. But they have no idea how to actually operate a network. And unless you try really really hard, it's too hard to understand networking to know what software is necessary. At the same time, network engineers are **TERRIBLE** at describing what they need. Actually, I think they don't know what they need, they ask for things that seem to make life more complicated, not less complicated overall. Most people, especially engineers, are bad at asking for what they need. They usually ask for a specific solution, when they don't understand the other field so they don't understand the tradeoffs being made there. Instead if they **described the problems that they had rather than asking for a better solution** I think we'd make a lot of progress.


## A systematic look at networking -- or is this it's own post



**The devices are the least interesting part of the network, but they get all the attention**. We need to be systematic and understand the whole network as a system, not just one particular piece. It's not that devices aren't interesting, I care about ASICs, and I care about protocol stacks, etc. I fall into this as well, I recently spent a week building benchmarking of different open source BGP stacks. However, these main building blocks are very complicated so the interactions of all the pieces throughout the network and how they are interact is critical to understand, and right now it's too hard to understand how they interact especially as things change, and there is always change.


As mentioned above, we focus too much on individual devices. What does the BGP configuration and policy need to be on this device? What we really need to be thinking about is how the whole network will behave. I think that's too hard right now, because we have to way to express system wide policy, we have no way to model what the whole network will do. It's easy to lab up with a single device (or maybe 2) to see if a configuration works. But you are missing how the whole network will work together.

We need a better approach to the network as a system.

## Examples of network engineering

Engineering is the art of making tradeoffs which means I want to be able to ask a bunch of questions. How can I do that now? I don't know anything about real engineering disciplines, but I'd bet that they do a lot of what-if scenarios and simulations. In networking the best we can do is great engineers on a whiteboard arguing about what will happen. This is very useful, and it works, but it's insufficient with how important networks. There's no technical reason that we can't make our decision with much better data, including numbers some times.

Here are some of the kinds of questions I'd like to be able to ask:

### Design time

I'd like to try out different routing protocols in my three tier clos. Facebook and Microsoft say that BGP is better than OSPF, [I beg to differ](https://elegantnetwork.github.io/posts/What-Ive-learned-about-OSPF/), but wouldn't you like to be able to actually try out the differences for yourself? Wouldn't it be nice to be able to model up a large network with either BGP or OSPF (or both), and then be able to simulate that? There are simulation systems, like GNS3, but it's generally too hard to build all the configuration to do that. How will convergence be affected by these choices? Not just generally, but with numbers, actually measured, so that you can compare.

I can't understand how my network will handle failure if it's large and has lots of different interactions. I need to understand how device, interface, and even protocol failures could cause me problem. At design time it would be nice to see what happens during failure.

These are the kind of complex routing policy changes that I need to make. Are they likely to be safe? Can this network in general handle these type of changes?


### Operations

This data that comes from design tools is the foundation for operations.If you record all your assumptions then you should be able to derive **ALL** of your thresholds from this data. Because it's tied to the design it is related to everything else


## Automation isn't enough

Most network engineers think that automation is the way to make networks better. And it is better than not, but it isn't enough. I don't believe automation as is usually practiced makes understanding the network better. It just makes the mechanical processes done by humans done by computers, which can easily make the network harder to understand if not done well, if not done systematically.

We need a much better way of thinking about the network, and much better abstractions to be automated against.

We do need to automate specific changes, but that's very difficult to do well over time without being able to describe how the whole network interacts and our expectations. From that should come specific changes. But until you have describe and modeled the assumptions, intent, etc. of the network then a human has to.

I don't mean to stop automating. I think the way we currently automate is missing a bunch of fundamental information, tools, and infrastructure which makes it hard to do well. It's too ad-hoc. So keep automating if it's all we have, but at the same time with better design tools we can make network automation better, safer, faster, etc.

Some people are using Source of Truth (SOT) systems for their automation, and this is sort of that direction. I think you need to be able to write up the rules for what is in the SOT, and not have to hand manage the data

## What should this look like?
network wide models

I want the design doc, that gets reviewed at the design meeting, to be the thing that actually produces the configuration.

correct by construction

[Get the assumptions out of our head](https://elegantnetworks.org/posts)


As mentioned, I think you have to start with the foundational information, which is design tools. From that you need to capture all the important data about a network.A way of describing high level design intent, assumptions, decisions, and from that be able to create device configuration for any device. We need the centralized control that SDN promised, but not necessarily in a centralized controller. You want to configure and manage everything together centrally, but you can still have the same distributed routing system.

Routing protocol policy in specific needs to be expressed as a centralized policy. If you can describe what you want from the whole network, then from that it should be fairly straightforward to produce the correct policy on each device.

I want the ability to make my own reference design. I want to be able to understand the tradeoffs in that design. ANd then I want to be able to create instances of that design, and make changes to it over time.

When making changes I need to have enough information about what we assume is true about the network and if there are violations of that.

## Current Solutions

There might be some good answers some of the networking companies, I haven't had a chance to try them all out.

what companies are really changing the way that networking is done? Not in the OpenFlow like way, which would never work, nor in the I don't actually understand the problem but I'm going to do something hypey. Probably not SDN. Controllers are a good idea for some problems, but most companies that talk about SDN are not solving real problems.

I think [Apstra](https://apstra.com/) might be working on this, but I don't know in detail how they work. I've never actually used Apstra, and the only time I talked to them was when I was at Amazon, and they couldn't scale and at least at the time didn't allow us to really get into the details of describing a design ourselves. I'm afraid that Apstra is too cookie cutter and doesn't really allow me to be able to specify the architecture that I want at a high level.

I'm also curious about [Gluware](https://gluware.com/), [Itential](https://www.itential.com/), [Anuta Networks](https://www.anutanetworks.com/). At some point I need to dive into [Cisco NSO] (https://www.cisco.com/c/en/us/products/cloud-systems-management/network-services-orchestrator/index.html), but I don't trust Cisco. I know that they bought the company, but still, it's so very hard to trust Cisco software. And I've never tried out [Arista's CloudVIsion](https://www.anutanetworks.com/), if it's not multivendor I think it's useless. Both because I can't imagine wanting to be locked into a single vendor, but also because if you don't have to do mutlivendor then you don't do a good job of really modeling the network.

I'm suspicious of all of these, I doubt that they are what we need, but I haven't tried them enough. Usually things like this focus way too much on visual and drag-n-drop. That's not the real problem. The real problems are that I can't even describe my design expectations and assumptions and that I can't model based on those things.