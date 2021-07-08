---
layout: post
comments: true
author: Justin Pietsch
title: Is my Network Healthy?
excerpt: Do you ask "Is my network healthy?" Do you know how to answer that well?
description: Knowing if your network is healthy or unhealthy is hard to do but critical in operating a well run network. It will help focus on what is important to making a better network that is more available for your customers.
---
Do you ask **"Is my network healthy?"** Do you know how to answer that question well? This is a critical question, but a hard and confusing one to answer well. It's also one that I think most people don't explicitly ask and should. As a network engineer, what does it even mean that the network is healthy? And if I figure that out, how do I answer that question? I probably don't have the systems to answer that question well. Is it my responsibility? It sure would be good if we as an industry had a better way to know that our networks are healthy.

It's very difficult, maybe impossible, to simply describe network health in a complete way. Or at least to describe it in a simple way that is measurable and actionable and covers every way you might have problems in your network. Sometimes you know your network is not healthy when a customer calls up and says "My application isn't working, I've checked everything, I think the problem is the network." How do you answer this question if your CIO asks you "Is the network working correctly?" or worse "How many times this month has the network been unhealthy?" It's very hard to prove to your self and the rest of your company that the network is completely healthy and not causing any problems.

Because it's so hard to completely define what a healthy network, when asking if the network is broken, what are you trying to accomplish? There are many things you want to know, some of which are practical and near term and some strategic. You want to know your network is broken so that you start fixing it. You want to be able to act on the data. Of course if you can measure if your network is healthy or unhealthy, then you also might want to track it over time to see if you are getting better at operating your network. This can be very important because then you can see which things give you better availability and which do not. Focusing on network health and availability will force you to give you a better service for your customers.


**Healthy doesn't just mean that there are no outages or no current customer complaints**, though those are very important. It includes "Is the network able to handle whatever changes happen?" That might be "I need to make a change, is the network healthy enough to do this change?"  Or worse, a junior engineer who's been here for six weeks needs to make a change. Or it might be "I need to add capacity, can we do that safely?" or upgrade the OS on all devices safely, or add new application traffic. It certainly is "Will the network be okay if we have a failure?"

Knowing if your network is healthy or not to a high degree of confidence is difficult, complicated, and never ending. Of course it would be nice if we could just take a whole bunch of measurements and turn it into a number and then say "If it's under 75, then it's broken." But it's never that easy for something as complex and important as a network. It is worth the effort. It is important. Just like testing software, you have to start and you keep iterating. Forever.

## Definition of Healthy Network

Thought it's hard, we need to have some definition of a healthy network. We'll define it loosely as the ability for applications to be able to do what they want to do without the network causing trouble. That's probably good. The problem is that it is hard to measure. How do you measure from all applications, especially if you are something like EC2 and customers might be doing anything?

It's often easier to know if your network is unhealthy. Sometimes it's obvious when everything is broken. Sometimes it's very subtle. The only way to really know if your network is healthy is the absence of unhealthy. In other words, if we don't know it's broken than it must be working fine. In some ways, that's all we can do. If that's so, then **we must be really confident that we have good ways to know when our network is unhealthy.**

It's also not enough to say every failure means that the network is unhealthy. If you have redundancy throughout your network and lose a device then the network is is still healthy, so you also need to have ways to understand all the various events that you might receive and figure out which are the important ones to tell you that the network is unhealthy.

Another important related question "Is my network behaving as expected?" This one can be even more subtle and complicated. It's important to think about because your network might be currently working correctly for all it's users, but not what you expect, so when things break or fail, you will have a disaster you weren't expecting.

## Measuring Health

Everything starts with Data. Data isn't everything, but you can't measure or act on network health without data. Different data allows different decisions.

I'd guess most networking organization don't explicitly think about measuring if their network is healthy. They start instead by thinking about what they can monitor and how to know that things have failed. But of course just because something has failed doesn't mean that's a problem for the network. How do I decide what's a real problem? There are also problems in the network that aren't as obvious as a device or link down that we also need to know about. This isn't sufficient. It doesn't cover all the things that could fail in the network, the lack of faults doesn't mean you are okay. What else needs to be covered? The standard set of SNMP metrics is not enough to trully understand your network health.

After you get all the data, then you have to translate it to an understanding of health. If my network is small I can just assume every device down is a pageable event. Every link down needs to be fixed as quickly as possible. But if your network gets big, that's not tenable. And people like to sleep, it's weird, so paging me for a device down when everything is okay is demoralizing. This isn't a good long term situation.

### All my data sources

What do we need to know if our networking is healthy or unhealthy? We need a lot of different sources of data. We need device data, up-down data, and event/syslog/snmptrap data. But we also need to be able to figure out which data is important and which events are important. **We have to have broad measurement to catch any outage, but also we need to be precise so that we can fix things quickly.**

There are a lot of data sources in the network, some of them are harder to make useful than others. Syslog might be critical, but hard to make useful. I have experienced several very bad outages in which there was a message in syslog to notify us, but we'd never seen it before so we didn't know to look for it. **I wish software engineers for NOS vendors were oncall for their own products, I think things like that would happen less.**

I know that I need data from several different sources to know if my network is healthy or not right now. I want to start with the obvious measuring device down, interface down, and standard interface monitoring, usually by SNMP.  Next I need to add event monitoring via SNMP traps and/or Syslog. This is harder to deal with because it's a lot of information and most of it doesn't tell me that something important has broken. It's usually important for going back and finding out what happened after an outage. After you learn which events give you the most information, then of course you can add those events as alerts.

You might have too much data already. It's hard to sift through all the data that you have to make meaningful decisions.

You also know that there is more data that you need. In most networks there is no monitoring for operational state. Are all the BGP sessions that you expect to be up up? Or OSPF neighbors, or eVPN state, etc.? If you don't measure all the important state, you can't know that things have failed and it's harder to be precise and actionable.

To do this all really well, you need to think about all the things that can fail, and make sure you have a way to monitor them all. Then you need to be able to figure out how to turn that data into figuring out what's important.

List of data possible sources:
- Fault monitoring -- SNMP or gNMI
- Performance -- SNMP or gNMI
- Event Monitoring -- SNMPtraps and Syslog
- Active Monitoring
- Operational State Data -- Suzieq
- Capacity management -- performance monitoring with long term trends
- Be able to check more than just standard metrics
	- Are all the necessary BGP peers up?
	- Do you have the right routes in the right places? Canary checking.

What other capabilities are necessary
- Good tickets prioritization
 
### Do you need active monitoring

Since we defined a healthy network as one that is not affecting application traffic, then creating active application traffic and measuring that seems a good idea. Even with instrumenting all of your devices and getting all the data available, you might have an outage that you don't know about. Part of it is you might not know how to instrument everything and part is that there is more data than you can make sense of.

Active monitoring or pingmesh is a really great data source. However, I'm not aware of any good active monitoring solutions that are widely available either commercially or open source. Active monitoring can be done in degrees. Some people still smokeping to actively monitor latency and loss of very particular IP addresses, usually to monitor WAN links. Inside a datacenter you can do active monitoring to try to find out when there are errors in your network that your others systems aren't telling you about. Very advanced active monitoring can actually pinpoint which devices in the network are misbehaving. To do that can be a huge amount of data and is a very hard task. I've seen a great system at Amazon, and it took years to develop. This is an area that I wish had good tools.

Active monitoring can be tricky because it can be noisy. If you already know you have a problem, it is great for isolating the problem. If the problem is remote site monitoring, usually the loss rate is so high it's easy to see. But in a datacenter with .5% packet loss affecting customers, or a small number of prefixes that are null routing, it's very hard to make an active monitoring system that can be precise and accurate.

Ideally high quality active monitoring would be widely available, but it isn't. This doesn't mean you can't be responsible to know if your network is healthy or unhealthy, but it does mean you won't always know right away and you will not have actionable precision all the time.

### Monitoring Challenges

There are many challenges to network monitoring. My number 1 issue is that network device vendors don't take monitoring seriously. I used to ask vendors if they had a counter for every possible packet drop. I've not gotten a good answer. At one meeting, the hardware people said "Of Course!" and the software people said: "We do?". It's actually hard to present those counters in a useful way and not make mistakes like over-counting. However, it doesn't matter if it's hard, we have to convince everyone that it's required.

Similarly, it's clear from network device syslog that whenever developers find errors they put error messages, but that doesn't mean the messages are useful, actionable, or organized in any sane way. There's a difference between writing in my code that I found an error so I'll syslog, vs I need to be able to understand all syslog messages and translate that into something a human can understand and be able to act on. This is very difficult, I agree, but I'd appreciate more effort.

**Grey failures are especially hard.** I think active monitoring is great way to do this, but it is really really hard to do well. Even with active monitoring, having active monitoring that covers everything is almost impossible. Have you ever experienced an outage in which a small number of devices filled up their ECMP group tables? Even with active monitoring you'd have to know that for some prefixes for some neighbors they weren't using all their neighbors. Very hard to monitor with active monitoring.

The real number 1 problem is apathy. If you think it's too hard to do, or it's not that big of a deal you want try to do this well, because it's very hard and you can never be perfect. But a well run network requires you to know very well if your network is healthy or unhealthy.

## Alerts, Events, Alarms

I've gotten an alert, is it important? I didn't get an alert, are things okay? Something has failed, is that a problem?

One of the big issues with network health is that you get all kinds of signals from your data sources, but they aren't all important. If a link fails, is that bad? If you get errors from an optic, is that bad? A device reboots, is that bad? All of these questions require the context of the network and how the rest of the network is behaving. If you have only two routers of some kind, and one crashes, then that is bad. You can't afford to have the other one fail. But if you have 24 and the other 23 are working fine then everything is all right for now.

You also don't want to wake some up for every fault, because it might not be important and you can't afford to burn our your operations team on unimportant problems. It's  not okay to be waking up people when there isn't really a problem. It demotivates people and you miss important events

- Are the alarms I'm getting important and prioritized? Do I get woken up for something not important? When I get woken up, can I make sense of all the different information I get?
- When I get an alarm is it easy to action? Does it tell me where to look or is to to general?

### Describing what you expect

There are also a set of questions around if the network is as expected. If you just notice that the network is down or negatively effecting customers that's the symptoms. What if your network is sick, but you don't know it, and you want to prevent an outage before it happens? This requires being able to describe what you expect your network to be. We can call this preventative health.

Another reason why this is important is that by describing what you expect you give your monitoring, alarming, and alerting services better data so that they can know when it's important to you. For instance, it would be nice to say that for this layer of the network, in this datacenter, I need to make sure that 3/4 of my devices (or interfaces or BGP sessions, etc.) are up at all times. In that way, you've given your alarming system better metadata about your intentions and assumptions about what matters.

## I need more tools

Once I'm thinking about how to really know that my network is healthy or unhealthy, I realize I need even more tools. Knowing if your network is healthy or not is critical, the next steps are knowing what is broken, why it's broken, and how to remediate.

If I now know when my network is healthy, I want to troubleshoot more quickly and more systematically. I want to be able to do great root cause analysis to find what caused the problem and prevent future outages. And then I need to know I can make safe changes if the Root Cause analysis found something that needs fixing.

## How to do this with Suzieq?

We've been working on Suzieq for [Network Observability](https://elegantnetwork.github.io/posts/observability/).  Right now it has fault information like device/interface/session up/down and we have a status page in the GUI with count of things that are up and down. Over time it will get more features like an alarm page.

Suzieq collects operational state data, which I think a lot of networks don't monitor. This allows more precision about what has failed.

Suzieq also has asserts, in which we have described rules that need to be true for correctness. It has rules like the MTUs of interfaces connected to each other need to be the same, and you can't have overlapping OSPF router IDs. This helps find and prevent problems by having describing some things about your network that you know should be correct. With the data that Suzieq has, or similar tools, what you want to be able to do is check all of the assumptions about health.

## Summary

No matter what you are doing or how you think about it, understanding your network health and continually striving to measure it better is critical to a well operated network.

1. Asking if your network is healthy is a critical question for your network.
1. Healthy is more than just that you aren't currently affecting customers. Do you have failures you don't know about that you could prevent from turning into customer problems?
1. Just monitoring Faults, like device or interface up/down is not enough!
1. Is your network behaving as expected?
1. Make your network as easy to measure as possible.
1. Monitor all the things that can fail.
1. Figure out how to aggregate and corelate alarms.
1. Keep innovating to make your ability to know if you network is healthy or unhealthy better.

## Suzieq

Try out [Suzieq](https://www.stardustsystems.net/suzieq/), the open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.

## Conversation

1. Do you have a good way of describing if your network is healthy or not?
2. If you think about knowing, at all times, if your network is healthy or unhealthy, how will that change your tools and the way that you operate  your network?
3. Which of these problems do you have? Do you have enough data, but you don't have good alarm aggregation, filtering, etc.?
tering, etc.?
work.

## Conversation

1. Do you have a good way of describing if your network is healthy or not?
2. If you think about knowing, at all times, if your network is healthy or unhealthy, how will that change your tools and the way that you operate  your network?
3. Which of these problems do you have? Do you have enough data, but you don't have good alarm aggregation, filtering, etc.?
ch of these problems do you have? Do you have enough data, but you don't have good alarm aggregation, filtering, etc.?
