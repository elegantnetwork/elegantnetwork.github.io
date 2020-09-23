---
layout: post
comments: true
author: Justin Pietsch & Dinesh Dutt
title: Helping Understand Distributed Systems
excerpt: 
description: 
---

Distributed systems are hard. Building them is hard. Deploying them is hard. Maintaining them is hard. Part of what distributed systems hard is that there are so many moving parts. Understanding how the different parts interact can be tricky. Try running gdb on a distributed system. Networking has been hard partly because a network is a distributed system, by default. 

One way to make sense of a distributed system is to observe its internal state. The state of a BGP session and the routing table are examples of what we mean by an internal state. If we can understand the internal states of each of the distributed pieces, there is a possibility we can understand what is going on with the system as a whole. When the going gets tough, the tough get their tools. One way to make distributed systems easier is to build tools. Thus building tools to help with this understanding is important. In Wikipedia, observability is defined as a measure of how well internal states of a system can be inferred from knowledge of its external outputs. Thus observability tools have the potential to help us make better sense of a distributed system. And a network observability tool helps us make sense of a functioning network as a whole. And in doing so, help us build, deploy and manage networks better.

Dinesh has [written](https://www.amazon.com/Cloud-Native-Data-Center-Networking/dp/1492045608) and presented about network observability before. In this post, we take a different slant on the topic, examining the different lenses through which we examine the power of observability (or monitoring).

## Monitoring vs Observability

Before we proceed however, there is one point to touch upon: the difference between monitoring and observability. Monitoring is something most people have been doing for a while. In networking, people monitor the interface state and the device state commonly. So how does observability relate to monitoring. Is it just old wine in a new bottle or is it something else? [Many observabilists say that monitoring is part of observability](https://medium.com/@copyconstruct/monitoring-and-observability-8417d1952e1c). Some think monitoring and observability are totally separate, and some who continue to be confused by the distinction. When we (Justin and Dinesh) started working together, this difference in usage and the meaning of terms affected our discussions too. When we stepped back and tried to walk away from terms and think of what we were hoping this monitoring or observability could achieve, we became more aligned. We both agree that we need better tools to help us make sense of the distributed system that is the network. 

We both agree with what we wrote in the [Suzieq Readme](https://github.com/netenglabs/suzieq), "We define observability as the ability of a system to answer either trivial or complex questions that you pose as you go about operating your network. How easily you can answer your questions is a measure of how good the system's observability is. Whether you say that ability is obtained via monitoring or observability, we decided to stop arguing about. So, if we continue to use the term observability in the rest of the document, please put it down to us trying to be hip, rather than deciding one term is more apt at describing the problem than another. 

## What does observability mean for networking?

There's actually very little content about network observability so we get to figure out how to make it useful. 

In networking today, the answer to the question, "**when I want to figure something out in the network, where do I start?**" is always either looking at graphs, or logging into a device. If you especially don't know what to do next, logging into the device is just about the only answer. If I'm troubleshoooting, I have to guess where the problem is, and I log into that device. I run show commands to get the data that I think I need. If I'm really desperate I might have to look at syslog. I will likely have to log into multiple devices to corelate between them. I probably need to figure out what just changed. Did some configuration change, or something else happen in the network? If I'm not troubleshooting, but just looking for simple answers such as the different OS versions I have, or where does an IP address live, I may have to struggle even more on where to start. With observability, we believe we can change the answer to be something more powerful, more holistic.

**We think that one of the big observability problems in networking is that we think often about device-by-device**, rather than putting all the information together and viewing it as a system. This is especially true for control plane information. You end up asking device-by-device, and have to string together information yourself, either in your head or hastily written script. I think for network observability, what we want is to centrally collect all the data that you need to troubleshoot, or validate, or investigate your network, and then provide tools to allow you to ask any sort of question about that data. 

Networks have a wealth of data. Our problem is collecting them in a useful manner, one that works across multiple platforms, and then having systems that allow you to get the answers you need. The common practice we've observed over the years is to at least collect metrics, typically over SNMP. The metrics most often include interface metrics, system metrics such as temperature, power etc. In a vast majority of cases it often stops here. Some others collect logs centrally, and use packet sampling to collect flow information (sFlow). **We think the biggest missing piece, and the most important part of networking to focus on for observability is how the control plane state**. By control plane state we include everything from routing protocol state, to forwarding table state and other pieces of information such as interface MTU. 

## Network Observability in Pratice

There are many different questions that you want to be able to ask in networking, some of them are too hard or take too long, and some are not possible. In blog posts about observability there are often lists of questions as demonstration. I want to start with trying to classify some set of questions that I think are especially interesting around this space.

### Operations, Debugging, and Root Cause Analysis

What do I do when I don't know what to do? Usually I log into a device. There has to be a better way.

The first class of things that jumps out it operations and debugging. It can be very hard to debug problems in the network. First off, the problems are distributed, so triangulating to find where the problem is and collecting data from every device is hard. Also, there are so many interacting systems on each device, did you collect all the right information? And there's the issue of time, did you collect the data when things were broken, or only now? Can you see how things changed over time?

Network debugging is mainly done with show commands run manually by network engineers. This is a really terrible way, even though we all do it. It requires humans to remember how to tie together show commands. It gets harder if you have multivendor networks; does everyone in my organization know the commands to run on each device type to get the right information?

What if you had an issue because you had mismatched MTU somewhere, and then you wanted to inventory your network for matching MTU? How hard is that question to ask? Or there was a bug in a version of the NOS you use, and you needed to know how many of your devices had that version? Or you had an outage and wanted to know what changed in OSPF in your network in the last two hours.

There are endless questions in this space.

### Understanding and Onboarding

When a new network engineer comes to your company, how do they familiarize themselves with the network? Maybe you have a map that you've hand drawn. Maybe you have software that draws the map for your. I know I'm in the minority, but after a network gets to be medium size, maps are generally not that useful. They usually don't represent enough of the complexity. But they are a good place to start. But do you draw it manually? Does it include all your protocols and addresses?

Can new engineers ask questions like which routing protocols do we have, and where are they? How many devices do I have of each type? How many versions of NOS do I have?

### Understanding and Design Interaction

An overlooked part of network engineering is network design evolution, which could apply in your current network or when you are building a new one. Networks are rarely ever "done", they evolve. For instance, you've noticed that your route count is growing in your network, and you are getting concerned about total count, what can you do about it? What prefixes could you summarize and where? Is it easy for you to explore your route tables and understand these questions?

What about trying to understand if your OSPF design is working well? How is your convergence? As you add devices is your OSPF convergence getting worse? Are there failures that were particularly bad? Are there interfaces that are flapping a lot and possibly over-stressing OSPF?

### Validation

Often when we make changes in the network, we run some checks before the change, make the change, and then make some new checks in the network. Often the checks are to make sure that what changed is what you thought changed, but I think a better way to think about them is to come up with checks to make sure that network is working correctly. A lot of those checks could be generalized and then could run all the time. This helps ensure your network is always working as expected. In some ways this is like automatic testing of software.

## What are some good solutions in the network?

Of course just because vendors and tools aren't calling themselves network observability tools doesn't mean they can't help in this area. So we'll see if we can find some useful tools in this space. Can I ask any of these type of questions with what is already available? I have no experience using or even demoing any of these tools, but they do look interesting. 

You likely already have something to tell you if devices or interfaces are down. You probably also have something that graphs traffic, so you can look for congestion, or have the system alert you for congestion. But what about other operational issues?

There are some tools that want to help you with the mapping piece, and the ability to onboard people. These also help orient you in your network. I think they are more focused on drawing pretty maps, but some of these seem to have the data that you would want in an observability tool. I've never used these, but they looked interesting. 
* <https://ipfabric.io/> This software focused on a map interface to help you troubleshoot and understand your network. It looks like it keeps track of your network over time. It has other features as well. I don't think it really offers much around observability over maps, but I don't know.
* <https://www.netbraintech.com/> This focuses on automation, but gives you a nice map of your network.
* <https://cumulusnetworks.com/products/netq/> I have no experience with this, it looks like it only works on servers and Cumulus and has some interesting data processing and Visualization.
* <https://www.arista.com/en/products/eos/eos-cloudvision> Does this do everything? I don't know if this helps with observability because it does so many things and I've never tried it out. I'm suspicious of software from device vendors, and especially ones that solve all my problems, but I haven't look at this at all.
* Cisco probably has something, but I can't understand Cisco's marketing to tell me what actually solves my problems. 


## [Suzieq](https://elegantnetwork.github.io/posts/Suzieq/) 

Dinesh and I are building [Suzieq for network observability](https://github.com/netenglabs/suzieq), and this is exactly the kind of questions we are trying to answer and the problems we are trying to solve.  Some you can do with Suzieq, many you cannot. We are just getting started with Suzieq. I know is s a pitch for Suzieq, and I apologize, but since we wrote Suzieq to solve these problems it's my best answer to demonstrate what network observability could look like. This doesn't mean we got it right, we'd love to know what other ways to solve these problems.

Suzieq is made to be the tool you go to when you don't know what to do next. Instead of logging into your devices, you can just go to Suzieq and orient yourself about your network. One of the useful things you can do with Suzieq is find out what has changed over a specific timeframe. 

There's always more data to add, and we need help. Dinesh [wrote up a bunch of questions](https://github.com/netenglabs/suzieq/blob/experiments/docs/q-to-ask.rst) to help him frame what to build for Suzieq. There's a lot of those questions we can't yet answer, though there are others he wasn't thinking of then that it can answer.

## Comments

Let us know if I'm missing any good software that you think helps for network observability.

## References

* <https://en.wikipedia.org/wiki/Observability>
* <https://www.youtube.com/watch?v=sj7pmz5sutE>
* <https://thenewstack.io/monitoring-and-observability-whats-the-difference-and-why-does-it-matter/>
* <https://www.networkcomputing.com/data-centers/what-observability-and-how-can-it-help-your-operations>
* <https://blog.twitter.com/engineering/en_us/a/2016/observability-at-twitter-technical-overview-part-i.html>
* <https://orangematter.solarwinds.com/2017/09/14/monitoring-isnt-observability/>





