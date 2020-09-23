---
layout: post
comments: true
author: Justin Pietsch & Dinesh Dutt
title: The Many Uses of Network Observability
excerpt: 
description: 
---

Distributed systems are hard. Building them is hard. Deploying them is hard. Maintaining them is hard. Part of what distributed systems hard is that there are so many moving parts. Understanding how the different parts interact can be tricky. Try running gdb on a distributed system. Networking has been hard partly because a network is a distributed system, by default. 

One way to make sense of a distributed system is to observe its internal state. The state of a BGP session and the routing table are examples of what we mean by an internal state. If we can understand the internal states of each of the distributed pieces, there is a possibility we can understand what is going on with the system as a whole. When the going gets tough, the tough get their tools. One way to make distributed systems easier is to build tools. Thus building tools to help with this understanding is important. In Wikipedia, observability is defined as a measure of how well internal states of a system can be inferred from knowledge of its external outputs. Thus observability tools have the potential to help us make better sense of a distributed system. And a network observability tool helps us make sense of a functioning network as a whole. And in doing so, help us build, deploy and manage networks better.

Dinesh has [written](https://www.amazon.com/Cloud-Native-Data-Center-Networking/dp/1492045608) and presented about network observability before. In this post, we take a different slant on the topic, examining the different lenses through which we examine the power of observability (or monitoring).

## Monitoring vs Observability

Before we proceed however, there is one point to touch upon: the difference between monitoring and observability. Monitoring is something most people have been doing for a while. In networking, people monitor the interface state and the device state commonly. So how does observability relate to monitoring. Is it just old wine in a new bottle or is it something else? [Many observabilists say that monitoring is part of observability](https://medium.com/@copyconstruct/monitoring-and-observability-8417d1952e1c). Some think monitoring and observability are totally separate, and some who continue to be confused by the distinction. When we (Justin and Dinesh) started working together, this difference in usage and the meaning of terms affected our discussions too. When we stepped back and tried to walk away from terms and think of what we were hoping this monitoring or observability could achieve, we became more aligned. We both agree that we need better tools to help us make sense of the distributed system that is the network. 

We both agree with what we wrote in the [Suzieq Readme](https://github.com/netenglabs/suzieq), "We define observability as the ability of a system to answer either trivial or complex questions that you pose as you go about operating your network. How easily you can answer your questions is a measure of how good the system's observability is. Whether you say that ability is obtained via monitoring or observability, we decided to stop arguing about. So, if we continue to use the term observability in the rest of the document, please put it down to us trying to be hip, rather than deciding one term is more apt at describing the problem than another. 

## What Does Observability Mean for Networking?

There's actually very little content about network observability so we get to figure out how to make it useful. 

In networking today, the answer to the question, "**when I want to figure something out in the network, where do I start?**" is always either looking at graphs, or logging into a device. If you especially don't know what to do next, logging into the device is just about the only answer. If I'm troubleshoooting, I have to guess where the problem is, and I log into that device. I run show commands to get the data that I think I need. If I'm really desperate I might have to look at syslog. I will likely have to log into multiple devices to corelate between them. I probably need to figure out what just changed. Did some configuration change, or something else happen in the network? If I'm not troubleshooting, but just looking for simple answers such as the different OS versions I have, or where does an IP address live, I may have to struggle even more on where to start. With observability, we believe we can change the answer to be something more powerful, more holistic.

[Can we say one of the biggest problems in networking...]
**We think that one of the big observability problems in networking is that we think often about device-by-device**, rather than putting all the information together and viewing it as a system. This is especially true for control plane information. You end up asking device-by-device, and have to string together information yourself, either in your head or hastily written script. I think for network observability, what we want is to centrally collect all the data that you need to troubleshoot, or validate, or investigate your network, and then provide tools to allow you to ask any sort of question about that data. 

Networks have a wealth of data. Our problem is collecting them in a useful manner, one that works across multiple platforms, and then having systems that allow you to get the answers you need. The common practice we've observed over the years is to at least collect metrics, typically over SNMP. The metrics most often include interface metrics, system metrics such as temperature, power etc. In a vast majority of cases it often stops here. Some others collect logs centrally, and use packet sampling to collect flow information (sFlow). **We think the biggest missing piece, and the most important part of networking to focus on for observability is how the control plane state**. By control plane state we include everything from routing protocol state, to forwarding table state and other pieces of information such as interface MTU.

## How is Network Observability Different From Compute Observability?

Transplanting an idea from one domain into another often requires tweaking the idea to work in the new environment. In compute, where the term originally came up, the [three pillars of observability](https://learning.oreilly.com/library/view/distributed-systems-observability/9781492033431/ch04.html) are metrics, logs, and distributed tracing. Distributed tracing applies to tracing function calls across various processes. For example, they often trace a customer web request all the way through the system. The equivalent of distrbuted tracing in networking might be flow monitoring (or s-flow), and SolarWinds [makes the case for that](https://blog.sflow.com/2019/10/observability-in-data-center-networks.html) [as do others](https://www.youtube.com/watch?v=sj7pmz5sutE). We think they make good and important points, but we don't think that is the most important part of observability for networking. Flow monitoring is like distributed tracing in distributed systems, but it is importantly different, and so not as important in networking. Today, in most networks, packets passing through a device don't get timestamped with important data and tags. Furthermore, packets can get dropped due to normal conditions and shouldn't be considered an error. We think control plane state is far more important to track than sflow state in many more cases.

## The Many Uses of Network Observability

In writings about observability there are often lists of questions as demonstration. Most of these lists are framed within the context of troubleshooting a problem. However, we think an observability tool can be used for many other useful things. SO, we'll examine the different use cases in which a good observability tool will be able to help, starting with the most obvious. The order of these things is not prioritized.

### Operations, Debugging, and Root Cause Analysis

The most obvious and commonly talked about use case is operations and debugging. It can be very hard to debug problems in the network. With the inherently distributed nature of the problem, the lack of a good observability tool hinders how quickly we can triangulate to find the source of the problem. In many cases, the relevant data is not even collected. In the few cases that it is collected, its not multi-vendor friendly. And even in the small set of cases where this is multi-vendor way--think interface stats or system stats such as CPU load and termperature--it is not possible to query the data in a meaningful way. Consider a common problem in low latency networks: corrupted packets spreading through the network due to cut-through switching. In such a scenario, we maybe collecting the relevant interface error counts, but its not easy to trace to the source of a problem because the most common way to represent all the information is via graphs on a dashboard. This is why they say you can't debug with a dashboard. Also, there are so many interacting systems on each device, did you collect all the right information? And there's the issue of time, did you collect the data when things were broken, or only now? Can you see how things changed over time?

Network debugging is mainly done with show commands run manually by network engineers. This is a really terrible way, even though we all do it. It requires humans to remember how to tie together show commands. It gets harder if you have multivendor networks; does everyone in my organization know the commands to run on each device type to get the right information?

What if you had an issue because you had mismatched MTU somewhere, and then you wanted to inventory your network for matching MTU? How hard is that question to ask? Or there was a bug in a version of the NOS you use, and you needed to know how many of your devices had that version? Or you had an outage and wanted to know what changed in OSPF in your network in the last two hours.

There are endless questions in this space. When What do I do when I don't know what to do? Usually I log into a device. There has to be a better way.

### Onboarding, Network Merges

Imagine joining a company as a network engineer. Or that you've just acquired a company and are tasked with merging the two networks. How do you familiarize themselves with the network? Spreadsheets, a few possibly outdated, inaccurate documents with a map of the network, and the memory and wisdom of older, more familiar engineers is the only way today. I know I'm in the minority, but after a network gets to be medium size, maps are generally not that useful. They usually don't represent enough of the complexity. But they are a good place to start. And a physical map of the topology is not the same as its routing topology or its overlay topology. 

Wouldn't it be easier if with a few commands you can understand how many routers there are, how many bridges, how many endpoints, how they're interconnected, the protocols used, the average size of the forwarding tables, a summary of subnet advertisments, the nnumber of VLANs and so on. This would be a far more useful, and self-exploratory yet deep way in which you can make sense of the network holistically. And then from here, you can drill down in any direction you please, see how things have changed (such as how often a software is patched) and so on. A good observability tool should be allow you to do these things.

### Understanding and Design Interaction

One of the customers we were dealing with wanted to migrate from a traditional vendor network to an EVPN-based network and wanted to know which vendor devices could handle the scale the numbers they had. They were a group of sharp individuals and seemed to know their stuff. But they over-estimated the number of MACs and routes they had by more than an order of magnitude, resulting in their concluding that the only available devices were the more expensive models. After we requested a careful review of a few key boxes, they concluded that their true numbers were much lower than their original estimation and so they could easily live with boxes with much smaller capacity even if they had growth in excess of their projections. If they had a simple tool that could tell them the current capacity of their forwarding tables, they could've arrived at the answer far faster, and without our requesting them for the review. 

In another case, a customer wanted to upgrade their OS from their current version because they needed a feature in the latest release. However, there was a critical bug that existed in upgrading one of the previous versions to the current version. They were fairly confident that this didn't affect them, but they ran into problems when they discovered that they had forgotten about a part of the network that still carried the troublesome older version. Again, a tool that could've listed the deployed OSes in their network would've helped them avoid this problem.

Other scenarios involve issues such as switching from OSPF to BGP. How can one understand if the OSPF design is working well enough? What is the current network convergence? As you add devices is your OSPF convergence getting worse? Is your NOS versions progressively getting more unstable? 

Good observability helps answer these questions too.

### Validation

When talking to network operators in the course of writing his book, Dinesh often heard the refrain that the operators were fearful of automating configuration because of the danger of a single mistake bringing down the network, thereby hurting their company's bottom line. Manual validation of the deployment is still the most common practice today. Furthermore, given the myriad complex ways in which vendors choose to display the output of show commands, stringing together an automated validation, post-deployment of configuration is a pretty challenging job. In other words, if you're barrelling down the infrastructure-as-code paradigm, how do you do the equivalent of automatic testing? A well designed observability tool should be able to help in this issue as well, for example by providing some snaity checking of state after configuration. A lot of those checks could be generalized and then could run all the time. This helps ensure your network is always working as expected. 

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





