---
layout: post
comments: true
author: Justin Pietsch
title: What is Network Observability
excerpt: Observability is an important concept in distributed systems these days. It's taken from control systems. ... How do we take these concepts from control systems and distributed systems and make them useful for networking?
description: What do we need to observe in a network? How is this different than monitoring? What specifically does it do?
---
## What is Observability
Observability is an important concept in distributed systems these days. It's taken from control systems, which have a very precise definition observability with a mathematical definition. It is a measurement of how much of the internal state of the system that can be observed by it's external information. In distributed systems it does not have a precise definition and things can get confusing as different people define it different ways. **The focus is on being able to ask arbitrarily hard questions and being able to get them answered in a timely manner**.

If observability is the property of a system in that you can understand it's state based on it's outputs, what does that mean for networking? I think it's also not enough that the data exists, but that you can answer questions in a timely manner. Networks have an immense amount of information available, from syslog, command outputs, flowspec, routing protocols, etc. How do you make use of that data? Do you have the data and systems to debug your network?

“Observability requires that you not have to predefine the questions you will need to ask, or optimize those questions in advance”  “We should be gathering data in a way that lets us ask any question, without having to predict in advance. “ “Observability requires interactivity, open-ended exploration” <https://www.honeycomb.io/blog/so-you-want-to-build-an-observability-tool/>


How do we take these concepts from control systems and distributed systems and make them useful for networking? What do we do with that? Can I already do the observability?

<maybe I don't care about this stuff -- maybe it goes away to get to the networking parts>

You have to design your system to be observable, meaning you have to provide access to data. In networks we actually have a wealth of data, our problem is collecting it in a useful manner and than having systems that allow you to answer the questions that you need.

Since I started talking to @Dinesh, he's brought up network observability. I'd never heard of that before. This is an important concept that I think isn't well understood, so I want to dive into it and see what I can find. In Dinesh's book he has a chapter on network observability. He and I don't completely agree about some of the classification (for instance, where monitoring fits in regard to observability), but the importance of being able to ask better questions of your network we agree on.

From our [Suzieq Readme](https://github.com/netenglabs/suzieq), "We define observability as the ability of a system to answer either trivial or complex questions that you pose as you go about operating your network. How easily you can answer your questions is a measure of how good the system's observability is. A good observable system goes well beyond monitoring and alerting." 

How is this different from monitoring? [Many observabilists say that monitoring is part of observability](https://medium.com/@copyconstruct/monitoring-and-observability-8417d1952e1c). Some think monitoring and observability are totally separate. I get confused. Amazon was doing some of this thing for their systems in 2000 and calling it monitoring. Some observabilists  think that observability has [three pillars: logs, metrics, and traces](https://learning.oreilly.com/library/view/distributed-systems-observability/9781492033431/ch04.html). [Some think that's nonsense](https://kccna18.sched.com/event/GrXI/three-pillars-zero-answers-we-need-to-rethink-observability-ben-sigelman-lightstep?iframe=no&w=100%&sidebar=yes&bg=no). There's a lot of taking words that already meant something, trying to more precisely define them, and then arguing that other people aren't doing it right. It's confusing. But the important thing is to try to figure out what is the problem statement that the observability camp is bringing up? 

## What does it mean for networking?
In the distributed system camp, they are often figuring out how to trace a customer web request all the way through the system. In networking the equivalent might be flow monitoring (or s-flow), and SolarWinds [makes the case for that](https://blog.sflow.com/2019/10/observability-in-data-center-networks.html) [as do others](https://www.youtube.com/watch?v=sj7pmz5sutE). I think they make good and important points, but I don't think that is the most important part of observability for networking, so I'll save this for another post. flow monitoring is like distributed tracing in distributed systems, but it is importantly different. Each packet through a device doesn't get timestamped with important data and tags, packets can get dropped an don't show up, etc. 

“Google's microservices generate about 5 billion RPCs per second; building observability tools that scale to 5B RPCs/sec therefore boils down to building observability tools that are profoundly feature poor. If your organization is doing more like 5 million RPCs/sec, that's still quite impressive, but you should almost certainly not use what Google uses: at 1/1000th the scale, you can afford much more powerful features”. <https://lightstep.com/blog/three-pillars-zero-answers-towards-new-scorecard-observability/>

What's the most interesting way to view this in networking? I think the most important part of networking to focus on for observability is how the control plane is doing. I'm being extra-generous with my idea of control plane here, I mean anything that is a control loop in the network system. For instance, anything that takes an interface out, or OSPF convergence, etc. I also mean collecting data like interface stats so that I can understand what's going on with the control plane.

There's actually very little content about network observability. I guess that means we get to define what it means!!! And figure out how to make it useful.

Or to say it another way, when I don't know how to figure something out in the network, where do I start? I guess where the problem is, and I log into that device. I run show commands to get the data that I think I need. If I'm really desperate I might have to look at syslog. I will likely have to log into multiple devices to corelate between them. I probably need to figure out what just changed. Did some configuration change, or something else happen in the network.

I think that one of the big observability problems in networking is that we think about device-by-device, rather than putting all the information together and viewing it as a system. That's how the protocols work, of course, but often it's not how we operate the network.

How does this relate to the monitoring you already have? You probably have interface monitoring or similar and that might be how you found out something was broken.

Of course just because vendors and tools aren't calling themselves network observability tools doesn't mean they can't help in this area. So we'll see if we can find some useful tools in this space.

Observability is multi-dimensional

In general in networking, monitoring starts with interface counters, usually provided by SNMP, as well as per device info like CPU utilization, temperature, etc. I think it often stops there, but some

an important set of data that are not collected systematically are show commands. These are the commands that provide network engineers the information that they need to operate and debug the network. Collecting these systematically gives you a lot of advantages: you have history over time, you can corelate output from multiple commands for better understanding, you can build alerting, etc.


what do I do when I don't know what to do? usually I log into a device. There has to be a better way

## Why does it matter -- Observability Questions
Being able to ask any question helps in many different areas in network
There are many different questions that you want to be able to ask in networking, some of them are too hard or take too long, and some are not possible. In blog posts about observability there are often lists of questions as demonstration. I want to start with trying to classify some set of questions that I think are especially interesting around this space.

### Operations and Debugging
The first class of things that jumps out it operations and debugging. It can be very hard to debug problems in the network. First off, the problems are distributed, so triangulating to find where the problem is and collecting data from every device is hard. Also, there are so many interacting systems on each device, did you collect all the right information? And there's the issue of time, did you collect the data when things were broken, or only now?

I'd guess that in almost every network, network debugging is mainly done with show commands run manually by network engineers. This is a really terrible way, even though we all do it. Too much requires humans to remember how to tie together show commands. It gets harder as you have multivendor networks; does everyone know the commands to run on each device type to get the right information.



What if you had an issue because you had mismatched MTU somewhere, and then you wanted to inventory your network for matching MTU? How hard is that question to ask?

There's almost endless questions in this space.

### Understanding and onboarding
When a new network engineer comes to your company, how do they familiarize themselves with the network? Maybe you have a map that you've hand drawn.

### Understanding and Design Interaction

A really important piece of networking is network design devolution, maybe in your current network or maybe because you are building a new one. Networks are rarely ever "done", they usually evolve. For instance, you've noticed that your route count is growing in your network, and you are getting concerned about total count, what can you do about it? What prefixes could you summarize and where? Is it easy for you to explore your route tables and understand these questions?

What about trying to understand if your OSPF design is working well? How is your convergence? As you add devices is your OSPF convergence getting worse? Are there failures that were particularly bad? Are there interfaces that are flapping a lot and possibly over-stressing OSPF?


## What are some good solutions in the network?
Can I ask any of these type of questions with what is already available? I have no experience using or even demoing any of these tools, but they do look interesting. 

There are some tools that want to help you with the mapping piece, and the ability to onboard people. These also help orient you in your network. I think they are more focused
* <https://ipfabric.io/> 
* <https://www.netbraintech.com/>

What about NetQ or Arista CloudVision?

Does Cisco have something useful?  I don't trust Cisco for their software solutions. I've never seen anything that came out from them that demonstrated they knew how to actually operate a network. 




## Suzieq -- or should this be it's own post?
Dinesh and I are building [Suzieq for network observability](https://github.com/netenglabs/suzieq), and this is exactly the kind of questions and the problems we are trying to solve with Suzieq. I'm going to talk about Suzieq as a way to demonstrate some of the things you want to do in this space. Some you can do with Suzieq, many you cannot. Come help us make it much better!!! There are many ways to help, one way is to join our slack and tell us the questions that you need answering. We are just getting started with Suzieq. 

One of the big things you can do with Suzieq is find out what has changed over a specific timeframe. 


We know there's a lot of things missing to make Suzieq a complete tool. We are currently focused on specific questions you can ask via a CLI (and soon a GUI). You can't easily ask questions that go across tables, unless you are willing to dive in and write Python and Pandas code. This isn't where we are currently focused but we'll get there.

Another important deficit is that we don't have good ways to add tags to aggregate data. We only have namespaces for grouping. If you'd like to know something about all the leaves, there's no way to do that right now. We'll add that soonish, it's too obviously important.

There's always more data to add, and we'll need help. Dinesh [wrote up a bunch of questions](https://github.com/netenglabs/suzieq/blob/experiments/docs/q-to-ask.rst) to help him frame what to build for Suzieq. There's a lot of those questions we can't answer, though there are others he wasn't thinking of then that it can answer.

## References
* <https://en.wikipedia.org/wiki/Observability>
* <https://www.youtube.com/watch?v=JiZPLnhz2L4>
* <https://www.youtube.com/watch?v=sj7pmz5sutE>
* <https://thenewstack.io/monitoring-and-observability-whats-the-difference-and-why-does-it-matter/>
* <https://www.networkcomputing.com/data-centers/what-observability-and-how-can-it-help-your-operations>
* <https://blog.twitter.com/engineering/en_us/a/2016/observability-at-twitter-technical-overview-part-i.html>
* <https://orangematter.solarwinds.com/2017/09/14/monitoring-isnt-observability/>