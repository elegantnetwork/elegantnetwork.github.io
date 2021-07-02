---
layout: post
comments: true
author: Justin Pietsch
title: Networking is Broken
excerpt: 
description: 
---

I think Network as an industry, discipline, and practice is broken, very badly broken. I don't know how to fix it, but I think it's critical to change how this industry works. I want to at least start talking about it. I don't even know how to describe all the things I think that are broken. 

I want a radically different industry, but I don't know how to describe it, much less find the people who can help build it. Obviously, there is a lot that is working since you know, the internet and networks rule the world. But I think we need much much better than we have to get the network out of the way

Better tools and better engineering. I think better engineering requires better tools. the word tools can get a bad rap because people build specific tools for a specific problem without thinking ahead and that's not what we are talking about here. There are pro


First off, the protocols aren't the problem. They work just fine enough. We have the pieces we need to build. Or at least, those aren't nearly as big a problem as that we don't have are systematic ways to design, build, scale, and operate networks. I honestly think that the individual router hardware, OS, and software are pretty low on what should be the focus, and they are almost the complete focus. There needs to be changes in the way that we think about networking and how we design, build, scale, and operate networks

THe devices are the least interesting part of the network, but they get all the attention. It's not that devices aren't interesting, I care about ASICs, and I care about protocol stacks, etc.

I'm demanding that network engineering be an engineering discipline, and not just people who read books from vendors and do what the vendor says. Engineering is about tradeoffs, you must know what tradeoffs you are making

There are probably plenty of places that going to Cisco and doing what they tell you is enough. But there are a lot of places in which that isn't enough but is still going on. (Using Juniper or Arista or ?? is a little better, at least you are thinking about some tradeoffs, but not much.)


It focuses too much on per device configuration and not on how the whole system fits together. It's not systematic enough, because there is almost no way to be systematic. 

One of the problems, of course is there are multiple dimensions. I want to just talk about IP connectivity. But of course there's all kinds of virtual networking for containers, Cloud computing, etc. Load balancing, firewalls, etc.


It's too hard to design, build, scale, and operate networks well. That is because we don't have the software systems around it that are necessary. I think the biggest missing piece is good design software and better design practices, but there are major missing pieces for all aspects of network engineering.

How do we deal with the complexity that is networking? How do we make good design decisions that solve the problems for the business that the network supports, as well as then is operatable over time? What does operatable mean: can scale appropriately, can make changes to, that I am confident are safe.

For instance when we were first designing the very large 3-tier Clos that makes up the aggregation network in the Amazon/AWS datacenters, some people were saying that recabling as we grow isn't that big a deal, and that datacenter techs aren't necessarily that expensive. True, however, recabling is a very tedious and potentially error prone process that is invasive and potentially very damaging. I tried to design a network that didn't require recabling. I didn't completely succeed because there were some scaling issues I couldn't deal with: I would have had to overbuild in many cases to avoid recabling in a couple cases.

However, Amazon/AWS has also built a sophisticated system for checking when cabling is done. 


How do I describe this. How do I describe what is required? Systems engineering. What doe that mean


I think design and architecture is not really given enough attention and instead operations and tinkering rule the day. Maybe


There are design and build problems and there are operations problems. I kind of want to separate them

I'm going to focus on the design side, because I think with good design tools everything gets better. Maybe you can't wait for good design tools and need better monitoring, observability, etc. That's also necessary, but an article for another day.

Why is design side so important? We need to be able to do engineering and study tradeoffs. After we know the network, we need to be able to describe our assumptions and assertion. From this, we can better validate and monitor the network. You can't validate that the network is working correctly if you can't describe how it should be working. Using config validation is wildly insufficient: how do you know that what you configured will do what you expect?


## What's wrong
I struggle to describe this coherently, and I've been noodling on it for so very long. My best explanation is analogy. Network Engineers and operators are like plumbers, but you wouldn't want plumbers architecting, designing, and building a city water works. 

Networking is different. The main building blocks (network devices like routers or firewalls) are much more complicated

Some part of it is the missing engineering. Some large part is that we don't have tools that allow a better way of system wide thinking. Some of it is that measuring and monitoring is still difficult and there is a lot of data.

I think there's a big problem with our hype cycle. Things get hyped that can't work out and waste a lot of time and energy for operators trying to guess what will work. 

How do we become better engineers? I think it's better tools and better approaches.


It's hard to design a network that is easy to build, scale and operate well. I don't think that when people are designing the network they are really thinking about those things. Not every place scales at the rate of AWS over the last decade, but you have to be thinking about what the changes are going to be.


## Examples of network engineering
I'd like to try out different routing protocols in my three tier clos


I can't understand how my network will handle failure.

Do I need to care about this current failure?


Is it safe to make this current change?

Do I have enough capacity to do maintenance?

I need to have a complex routing policy and I need to know that it's going to work the way I think it will work. It sure would be helpful if I could describe how I think it's going to work.


## Systems Engineering
There seems to me to be a engineering piece missing. 

It's the engineering approach that's missing and there are not tools that enable it.


Missing the global view, which is weird. Networking is about a whole network, but the focus of most discussion is per router configuration. There is no good way of describing 


## Automation isn't enough
https://www.evernote.com/shard/s3/nl/221926/05034f9e-dac3-422c-8fb6-3dd7a5e850eb?title=automation%20isn't%20enough 

We need a much better way of thinking about the network, and much better abstractions to be automated against.

THe problem isn't that we need to automate specific changes, it's that we need to be able to describe how the whole network interacts and our expectations. From that should come specific changes. But until you have describe and modeled the assumptions, intent, etc. of the network then a human has to 


## What are the consequences
lots of tinkering

THe network is not out of the way


## What can we learn from other disciplines

When the Intel 486 came out in 1989, I remember seeing a magazine cover with the chip on the front and the article describing 1Million transistors. The blew my teenage mind. We are now at the place in which there are networks with around that many devices. And routers are a lot more complicated than transistors.


Most networks don't have 1 M devices, but I think even if you have 10, or 100, and certainly by 1000 you should have real design tools.

## What is needed for all networks, large or small

### What do extremely large networks need


## What about design tools


## Monitoring and Observability
##  Solutions
THere might be some good answers in some of the latest startups, I haven't had a chance to try them all out.

what companies are really changing the way that networking is done? Not in the OpenFlow like way, which would never work, nor in the I don't actually understand the problem but I'm going to do something hypey. Probably not SDN. Controllers are a good idea for some problems, but most companies that talk about SDN are not solving real problems.

### Design Software

I think Apstra might be working on this, but I don't know in detail how they work. I've never actually used Apstra, and the only time I talked to them was when I was at Amazon, and they couldn't scale and weren't really further along than what we already had. I'm afraid that Apstra is too cookie cutter and doesn't really allow me to be able to specify the architecture that I want at a high level.

### monitoring and observability

Suzieq, but honestly Dinesh and I have a lot more ideas on what Suzieq could do than we can possibly build with just 2 people. 

Augtera, but I haven't looked at it, somebody I know thinks it's really interesting.




## What do I want built

### Design tools
A way of describing high level design intent, assumptions, decisions, and from that be able to create device configuration for any device.


We need the centralized control that SDN promised, but not necessarily in a centralized controller. You want to configure and manage everything together centrally, but you can still have the same distributed routing system.

routing protocol policy in specific needs a system wide

I want the ability to make my own reference design. I want to be able to understand the tradeoffs in that design. ANd then I want to be able to create instances of that design, and make changes to it over time.

when making changes
 * 




maybe SD-WAN approaches solve this for the regular enterprise WAN needs. What I don't know is how flexible any of these solutions are. How much of it do you really get to make design decisions? I've never used them, but they sound like they are focused on making a system wide decisions on how to make the network be useful. I 

## Conversation / Talk-back
WHo am I missing that is really trying to change networking?

I don't know everything. There might be solutions to some of the problems that I'm describing.