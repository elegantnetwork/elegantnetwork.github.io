---
layout: post
comments: true
author: Justin Pietsch
title: Is my Network Healthy?
excerpt: Do you ask "Is my network healthy?" Do you know how to answer that well?
description: Knowing if your network is healthy or unhealhty is hard to do but critical in operating a well run network.
---
Do you ask **"Is my network healthy?"** Do you know how to answer that well? This is a critical question, but a hard and confusing one to answer well. It's also one that I think most people don't ask explicitly and should. What does it even mean that the network is healthy? And if I figure that out, how do I answer that question? I probably don't have the systems to answer that question well. Is it my responsibility? We have to figure out what it means to know that your network is healthy. 

What are you trying to do with that question? One thing is to know your network is broken so that you start fixing it. But of course if you can measure if your network is healthy or unhealthy, then you also might want to track it over time to see if you are getting better at operating your network. This can be very important because then you can see which things give you better availability and which do not. Focusing on network healthy and availability will force you to give you a better service for your customers.

Sometimes you know your network is unhealthy when a customer calls up and says "My application isn't working, I've checked everything, I think the problem is the network." Sometimes it's when your CIO asks you "Is the network working correctly?" or worse "How many times this month has the network been unhealthy?"


**Healthy doesn't just mean that there are no outages or no current customer complaints**, though those are very important. I think it includes "Is the network trustworthy to do the things that we need to do with it?" That might be "I need to make a change, is the network healthy enough to do this change?" Or it might be "I need to add capacity, can we do that safely?" or upgrade the OS on all devices safely, or add a new application traffic.

If you think it's your responsibility to know if your network is healthy or not then you have to think deeply about the questions you care about and how you can answer those. 

Of course it would be nice if we could just take a whole bunch of measurements and turn it into a number and then say "If it's under 75, then it's broken." But it's never that easy for something as complex and important as a network. 

### Definition of Healthy Network

In truth it's very hard to define a healthy network. We'll define it loosely as the ability for applications to be able to do what they want to do without the network causing trouble. That's probably good. The problem is that it is very hard to measure. How do you measure from all applications, especially if you are something like EC2 and customers might be doing anything. (In truth, Amazon has a lot of random things going on, it's been impossible to know what all internal traffic will do.)

It's easier to answer "Is my network unhealthy?" than "Is my network healthy?", so we decide that our network is healthy because we don't know that it's unhealthy. In other words, if we don't know it's broken than it must be working fine. In some ways, that's all we can do. If that's so, then we must be really confident that we have good ways to know when our network is unhealthy.

It's also not enough to say every failure, or anything going done means that the network is unhealthy. If you have redundancy and lose a device, then the network is is still healthy, so you also need to have ways to understand all the various events that you might receive and figure out which are the important ones to tell you that the network is unhealthy.

There's another important related question "Is my network behaving as expected?" This one can be even more subtle and complicated. It's important to think about because your network might be currently working correctly for all it's users, but what you expect, so when things break or fail, you will have a disaster you weren't expecting.

## What are we trying to measure?

I'd guess most networking organization don't explicitly think about if their network is healthy. They start instead by thinking about what they can monitor and how to know that things have failed But of course just because something has failed doesn't mean that's a problem for the network. How do I decide what's a real problems? And there are also problems in the network that aren't as obvious as a device or link down that we also need to know about. 

We want to know when the network is broken and affecting customers, but we also need to know when anything fails so that we can prevent customer outages. So we are kind of trapped. We want to collect all possible information to tell us that things are broken in the network, but we have to have a way to deal with all the information to decide what is important.

If my network is small I can just assume every device down is a pageable event. Every link down needs to be fixed as quickly as possible. But if your network gets big, that's not tenable. And people like to sleep, it's weird, so paging me for a device down when everything is okay is demoralizing. This isn't a good long term situation.

### All my data sources

What do we need to know if our networking is healhty or unhealthy? We need a lot of data. We need device data, up-down data, event/syslog/snmptrap data. But we also need to be able to figure out which data is important and which events are important.

There are a lot of data sources in the network, and some of them are much harder to make useful than others. syslog might be critical, but super hard to make useful. I have experienced several very bad outages in which there was a message in syslog to notify us, but we'd never seen it before so we didn't know to look for it. **I wish software engineers for NOS vendors were oncall for their own products, I think things like that would happen less.**

I know that I need a lot of data to know if my network is healthy or not right now. I want to start with the obvious of measuring device down, interface down, and standard interface monitoring, usually by SNMP.  Next I need to add event monitoring via SNMP traps and/or syslog. This is harder to deal with because it's a lot of information and most of it doesn't tell me that something important has broken. It's usually important for going back and finding out what happened after an outage. After you learn which events give you the most information, then of course you can add those events as alerts.

There's more data that you probably need. In most networks there is no monitoring for operational state. Are all the BGP sessions that you expect to be up up? Or OSPF neighbors, or eVPN state, etc. If you don't measure all the important state, you can't know that things have failed. 

So you need to think about all the things that can fail, and make sure you have a way to monitor them all. Then you need to be able to figure out how to turn that data into figuring out what's important.

### Monitoring and Data
So what data sources should I be looking at:
- Fault monitoring -- SNMP or GNMI
- Performance -- SNMP or GNMI
- Event Monitoring -- SNMPtraps and Syslog
- Active Monitoring -- pingmesh
- Capacity management -- performance monitoring with long term trends
- Be able to check more than just standard metrics
	- Like are all the necessary bgp peers up. 
	- Do you have the right routes in the right places. Canary checking.

What other capabilities are necessary
- Good tickets prioritization
- Have to have the right Attitude. Is it more important to know right away or to not wake people up. But you also don’t want 
 

### Do you need active monitoring

Even with instrumenting all of your devices and getting all the data available, you might have an outage that you don't know about. Part of it is you might not know how to instrument everything and part is that there is more data than you can make sense of. 

Active monitoring, pingmesh, is a really great data source. Active monitoring can be done in degrees. Some people still smokepoing to actively monitor lateny and loss of very particular IP addresses, usually to monitor WAN links. Inside a datacenter you can do active monitoring to try to find out when there are errors in your network that your others systems aren't telling you about. Very advanced active monitoring can actually pinpoint which devices in the network are misbehaving. To do that can be a huge amount of data and is a very hard task. I've seen a great system at Amazon, and it took years to develop. I don't know any commercial or open source offering for pingmesh. This is an area that I wish had good tools.


### Challenges

There are many challenges. I don't think any network hardware vendor takes that kind of monitoring seriously. Though it's been a long time, I used to ask vendors if they had a counter for every possible packet drop. I've not gotten a good answer. At one place, the hardware people said "Of Course!" and the software people said: "". It's actually hard to present those counters in a useful way and not make mistakes like over-counting. However, it doesn't matter if it's hard, we have to convince everybody that it's required.


### grey failures are especially hard

I think pingmesh is great way to do this, but it is really really hard to do well.


I wrote about the network being in the way. Not knowing that the network has failed until a customer tells you means that you are in the way



### What about customer traffic

In some ways the only signal that matters is to know if customer traffic is okay. But since you want to know before customers complain you can't rely on the to tell you it's broken.

## Alerts, Events, Alarms

I've gotten an alert, is it important? I didn't get an alert, are things okay? Something has failed, is that a problem?

One of the big issues with network health is that you get all kinds of signals from your data sources, but they aren't all important. If a link fails, is that bad? If you get errors from an optic, is that bad? A device reboots, is that bad? All of these questions require the context of the network and how the rest of the network is behaving. If you have only two routers of some kind, and one crashes, then that is bad. You can't afford to have the other one fail. But if you have 24 and the other 23 are working fine then everything is all right for now.

You also don't want to wake some up for every fault, because it might not be important and you can't afford to burn our your operations team on unimportant problems. It's  not okay to be waking up people when there isn't really a problem. It demotivates people and you miss important events

- Are the alarms I'm getting important and prioritized? Do I get woken up for something not important? When I get woken up, can I make sense of all the different information I get?
- When I get an alarm is it easy to action? Does it tell me where to look or is to to general?

### Describing what you expect
There are also a set of questions around if the network is as expected. If you just notice that the network is down or negatively effecting customers that's the symptoms. What if your network is sick, but you don't know it, and you want to prevent an outage before it happens? This requires
being able to describe what you expect your network to be. I guess this would be preventative health. Or like you want to catch the cancer before there are 

Another reason why this is important is that by describing what you expect you give your monitoring, alarming, and alerting services better data so that they can know when to page you. For instance, it should would be nice to say that for this layer of the network, in this datacenter, I need to make sure that 3/4 of my devices (or interfaces or BGP sessions, etc.) are up at all times. In that way, you've given your alarming system better metadata about your intentions and assumptions about what matters.

## You still don't get it all right
There are times in which all your systems are telling you things are running fine, but it turns out you aren't asking the right questions. Not exactly related, but in 2006 I was responsible for load balancers at Amazon, and I walked by  our VP of Infrastructure in the hall who asked me if we were doing fine for capacity. I said yes because all our indications were that it was fine. Then I started thinking about it more, and went home and started looking. I realized that we (I) had been incorrectly modeling capacity, and that we were already out of capacity under specific failure. You will get this wrong and you need to be able to ask and explore your network so that you can discover new things that you didn't realize you needed to care about.


## How to do this with Suzieq
We've been working on Suzieq for Network observability. Suzieq isn't great for this yet, but we have ideas on how to make it great. Right now it nows fault information like device/interface/session up/down and we have a status page in the GUI with count of things that are up and down We will be adding an alarm page to keep track of which things are broken.

Suzieq also has asserts, in which we have described rules that need to be true for correctness. It has rules like the MTUs of interfaces connected to each other need to be the same, and you can't have overlapping OSPF router IDs. This helps find and prevent problems. Right now you must run each individal assert. We will add asserts to the alarm page.

There is so much more than can be done with the data that is in Suzieq for Network Health. Help us figure out what problems you most need help with.

## Summary
1. Healthy is more than just you aren't currently affecting customers. Do you have failures you don't know about that you could prevent from turning into customer problems?
1. Just monitoring Faults, like device or interface up/down is not enough!
1. Over-paging is as bad as under-paging.
1. Is your network behaving as expected?
1. Make your network as easy to measure as possible.
1. Monitor all the things that can fail.
1. Figure out how to aggregate and corelate alarms.
1. Keep innovating to make your ability to know if you network is healthy or unhealthy better.

## Suzieq
Try out [Suzieq](https://www.stardustsystems.net/suzieq/), our open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.

## Conversation
1. Do you have a good way of describing if your network is healthy or not?
2. 