---
layout: post
comments: true
author: Justin Pietsch
title: What is Network Observability
excerpt: Observability is an important concept in distributed systems these days. It's taken from control systems. ... How do we take these concepts from control systems and distributed systems and make them useful for networking?
description: What do we need to observe in a network? How is this different than monitoring? What specifically does it do?
---
## What is Observability

Observability is an important concept in distributed systems these days. It is taken from control systems, which has a very precise mathematical definition. It is a measurement of how much of the internal state of the system that can be observed by it's external information. In distributed systems it does not have a precise definition and things can get confusing as different people define it different ways. **The focus is on being able to ask arbitrarily hard questions and being able to get them answered in a timely manner**.

“Observability requires that you not have to predefine the questions you will need to ask, or optimize those questions in advance.” “We should be gathering data in a way that lets us ask any question, without having to predict in advance." “Observability requires interactivity, open-ended exploration” <https://www.honeycomb.io/blog/so-you-want-to-build-an-observability-tool/>

If observability is the property of a system in that you can understand it's state based on it's outputs, what does that mean for networking? It's also not enough that the data exists, but that you can answer questions in a timely manner. Networks have an immense amount of information available, from syslog, command outputs, flowspec, routing protocols, etc. In networks we actually have a wealth of data, our problem is collecting it in a useful manner and than having systems that allow you to answer the questions that you need. Do you have the data and systems to debug your network? How do we take these concepts from control systems and distributed systems and make them useful for networking? What do we do with that? You have to design your system to be observable, meaning you have to provide access to data. 

Since I started working with Dinesh Dutt, he's brought up network observability, which I'd never heard of before. In [Dinesh's book](https://www.amazon.com/Cloud-Native-Data-Center-Networking/dp/1492045608) Chapter 11 is on network observability. He and I don't completely agree about some of the classification (for instance, where monitoring fits in regard to observability), but the importance of being able to ask better questions of your network we agree on.

From the [Suzieq Readme](https://github.com/netenglabs/suzieq), "We define observability as the ability of a system to answer either trivial or complex questions that you pose as you go about operating your network. How easily you can answer your questions is a measure of how good the system's observability is. A good observable system goes well beyond monitoring and alerting." 

How is this different from monitoring? [Many observabilists say that monitoring is part of observability](https://medium.com/@copyconstruct/monitoring-and-observability-8417d1952e1c). Some think monitoring and observability are totally separate. I get confused about the distinction. Amazon was doing some of this thing for their systems in 2000 and calling it monitoring. Some observabilists think that observability has [three pillars: logs, metrics, and traces](https://learning.oreilly.com/library/view/distributed-systems-observability/9781492033431/ch04.html). [Some think that's not nearly enough for observability](https://kccna18.sched.com/event/GrXI/three-pillars-zero-answers-we-need-to-rethink-observability-ben-sigelman-lightstep?iframe=no&w=100%&sidebar=yes&bg=no). 

There's a lot of trying to precisely define words that already meant something, and then arguing that other people aren't doing it right. It's confusing. But the important thing is to think about if there are questions you can't ask your system, and how you would like to improve that situation.

## What does observability mean for networking?

There's actually very little content about network observability so we get to figure out how to make it useful.

I think the most important part of networking to focus on for observability is how the control plane is doing. I'm being extra-generous with my idea of control plane here, I mean anything that is a control loop in the network system. For instance, anything that takes an interface out, or OSPF convergence, etc. I also mean collecting data like interface stats so that I can understand what's going on with the control plane.

To say it another way, when I don't know how to figure something out in the network, where do I start? I have to guess where the problem is, and I log into that device. I run show commands to get the data that I think I need. If I'm really desperate I might have to look at syslog. I will likely have to log into multiple devices to corelate between them. I probably need to figure out what just changed. Did some configuration change, or something else happen in the network?

I think that one of the big observability problems in networking is that we think often about device-by-device, rather than putting all the information together and viewing it as a system. This is especially true for control plane information. You end up asking device-by-device, and have to string together information yourself, either in your head or hastily written script. I think for network observability, what we want is to centrally collect all the data that you need to troubleshoot, or validate, or investigate your network, and then provide tools to allow you to ask any sort of question about that data. 

You probably already have interface monitoring or similar and that might be how you found out something was broken. In general in networking, monitoring starts with interface counters, usually provided by SNMP, as well as per device info like CPU utilization, temperature, etc. I think it often stops there. This is important data to have, and you'd like to combine it with other information in your system to ask better questions. But what do you do next? Do you have tools in place to tell you why an alert happened? You probably will log into your devices and run show commands. These are the commands that provide network engineers the information that they need to operate and debug the network. Collecting these systematically gives you a lot of advantages: you have history over time, you can corelate output from multiple commands for better understanding, you can build alerting, etc.

Is there other data that would be helpful to collect in central place that would help you be able to understand your network? Sometimes you do need those SNMP interface stats to understand how traffic changed or that you are getting polarization in your ECMP traffic. Syslog is also a source of rich, but poorly formatted and hard to reason, about data. It's surprising how often I've seen a new syslog message that we'd never seen before during an outage. If you didn't know that those could happen, they are hard to catch. Syslog is a rich source of data, I've never seen it dealt with well in networking, but I certainly haven't looked at what is currently available.

In the distributed system camp, they often trace a customer web request all the way through the system. In networking the equivalent might be flow monitoring (or s-flow), and SolarWinds [makes the case for that](https://blog.sflow.com/2019/10/observability-in-data-center-networks.html) [as do others](https://www.youtube.com/watch?v=sj7pmz5sutE). I think they make good and important points, but I don't think that is the most important part of observability for networking. Flow monitoring is like distributed tracing in distributed systems, but it is importantly different, and so not as important in networking. Each packet through a device doesn't get timestamped with important data and tags, packets can get dropped an don't show up, etc. 

## Why does it matter -- Network Observability Questions

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

An overlooked part of network engineering is network design devolution, maybe in your current network or maybe because you are building a new one. Networks are rarely ever "done", they evolve. For instance, you've noticed that your route count is growing in your network, and you are getting concerned about total count, what can you do about it? What prefixes could you summarize and where? Is it easy for you to explore your route tables and understand these questions?

What about trying to understand if your OSPF design is working well? How is your convergence? As you add devices is your OSPF convergence getting worse? Are there failures that were particularly bad? Are there interfaces that are flapping a lot and possibly over-stressing OSPF?

### Validation

Often when we make changes in the network, we run some checks before the change, make the change, and then make some new checks in the network. Often the checks are to make sure that what changed is what you thought changed, but I think a better way to think about them is to come up with checks to make sure that network is working correctly. A lot of those checks could be generalized and then could run all the time. This helps ensure your network is always working as expected. In some ways this is like automatic testing of software.

## What are some good solutions in the network?

Of course just because vendors and tools aren't calling themselves network observability tools doesn't mean they can't help in this area. So we'll see if we can find some useful tools in this space. Can I ask any of these type of questions with what is already available? I have no experience using or even demoing any of these tools, but they do look interesting. 

You likely already have something to tell you if devices or interfaces are down. You probably also have something that graphs traffic, so you can look for congestion, or have the system alert you for congestion. But what about other operational issues?

There are some tools that want to help you with the mapping piece, and the ability to onboard people. These also help orient you in your network. I think they are more focused on drawing pretty maps, but some of these seem to have the data that you would want in an observability tool. I've never used these, but they looked interesting. 
* <https://ipfabric.io/> This software focused on a map interface to help you troubleshoot and understand your network. It looks like it keeps track of your network over time. It has other features as well.
* <https://www.netbraintech.com/> This focuses on automation, but gives you a nice map of your network.
* <https://cumulusnetworks.com/products/netq/> I have no experience with this, it looks like it only works on servers and Cumulus and has some interesting data processing and Visualiation
* <https://www.arista.com/en/products/eos/eos-cloudvision> Does this do everything? I don't know if this helps with observability because it does so many things and I've never tried it out. I'm suspicious of software from device vendors, and especially ones that solve all my problems, but I haven't look at this at all.
* Cisco probably has something, but I can't understand Cisco's marketing to tell me what actually solves my problems. 


## [Suzieq](https://elegantnetwork.github.io/posts/Suzieq/) 

Dinesh and I are building [Suzieq for network observability](https://github.com/netenglabs/suzieq), and this is exactly the kind of questions we are trying to answer and the problems we are trying to solve.  Some you can do with Suzieq, many you cannot. Come help us make it much better!!! There are many ways to help, one way is to join our slack and tell us the questions that you need answering. We are just getting started with Suzieq. I know this sounds a pitch for Suzieq, and I apologize, but since we wrote Suzieq to solve these problems it's my best answer to demonstrate what network observability could look like.

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