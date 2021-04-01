---
layout: post
comments: true
author: Justin Pietsch
title: Is my Network Healthy?
excerpt: Do you ask "Is my network healthy?"
description: 
---
Do you ask "Is my network healthy?" This is a critical question, but a hard and confusing one to answer well. It's also one that I think most people don't ask explicitly and should. What does it even mean that the network is healthy? And if I figure that out, how do I answer that question? I probably don't have the systems to answer that question well. Is it my responsibility? We have to figure out what it means to know that your network is healthy. 

What are you trying to do with that question? I think the most important is that you know your network is broken so that you start fixing it. But of course, if you can measure if your network is healthy or unhealhty, then you also might want to track it over time to see if you are getting better at operating your network. This can be very important because then you can see which things give you better availability and which do not. Focusing on network healthy and availability will force you to give you a better service for your customers.

Sometimes you know your network is unhealthy when a customer calls up and says "My application isn't working, I've checked everything, I think the problem is the network." Sometimes it's when your CIO asks you "Is the network working correctly?" or worse "How many times this month has the network been unhealthy?"

Healthy doesn't just mean that there are no outages or no current customer complaints, though those are very important. I think it includes "Is the network trustworthy to do the things that we need to do with it?" That might be "I need to make a change, is the network healthy enough to do this change?" Or it might be "I need to add capacity, can we do that safely?" or upgrade the OS on all devices safely, or add a new application traffic.

If you think it's your responsibility to know if your network is healthy or not then you have to think deeply about the questions you care about and how you can answer those. 

Of course it would be nice if we could just take a whole bunch of measurements and turn it into a number and then say "If it's under 75, then it's broken." But it's never that easy for something as complex and important as a network. 

### Definition of Healthy Network

In truth it's very hard to define a healthy network. We'll define it loosely as the ability for applications to be able to do what they want to do without the network causing trouble. That's probably good. The problem is that it is very hard to measure. How do you measure from all applications, especially if you are something like EC2 and customers might be doing anything. (In truth, Amazon has a lot of random things going on, it's been impossible to know what all internal traffic will do.)

It's easier to answer "Is my network unhealthy?" than "Is my network healthy?", so we decide that our network is healthy because we don't know that it's unhealthy. In other words, if we don't know it's broken than it must be working fine. In some ways, that's all we can do. If that's so, then we must be really confident that we have good ways to know when our network is unhealthy.

It's also not enough to say every failure, or anything going done means that the network is unhealthy. If you have redundancy and lose a device, then the network is is still healthy, so you also need to have ways to understand all the various events that you might receive and figure out which are the important ones to tell you that the network is unhealthy.

There's another important related question "Is my network behaving as expected?" This one can be even more subtle and complicated. It's important to think about because your network might be currently working correctly for all it's users, but what you expect, so when things break or fail, you will have a disaster you weren't expecting.

## What are we trying to measure?

I'd guess most networking organization don't really think about if their network is healthy. They start instead by thinking about what they can monitor and how to know that things have failed But of course just because something has failed doesn't mean that's a problem for the network. How do I decide what's a real problems? And there are also problems in the network that aren't as obvious as a device or link down that we also need to know about. 

We want to know when the network is broken and affecting customers, but we also need to know when anything fails so that we can prevent customer outages. So we are kind of trapped. We want to collect all possible information to tell us that things are broken in the network, but we have to have a way to deal with all the information to decide what is important.

If my network is small I can just assume every device down is a pageable event. Every link down needs to be fixed as quickly as possible. But if your network gets big, that's not tenable. And people like to sleep, it's weird, so paging me for a device down when everything is okay is demoralizing. This isn't a good long term situation.



### All my data sources

There are a lot of data sources in the network, and some of them are much harder to make useful than others. syslog might be critical, but super hard to make useful. I have experienced several very bad outages in which there was a message in syslog to notify us, but we'd never seen it before so we didn't know to look for it. **I wish software engineers and NOS vendors were oncall for their own products, I think things like that would happen less.**

I know that I need a lot of data to know if my network is healthy or not right now. I want to start with the obvious of measuring device down, interface down, and standard interface monitoring, usually by SNMP.  Next I need to add event monitoring via SNMP traps and/or syslog. This is harder to deal with because it's a lot of information and most of it doesn't tell me that something important has broken. It's usually important for going back and finding out what happened after an outage. After you learn which events give you the most imformation, then of course you can add those events as alerts.

There's more data that you probably need. In most networks there is no monitoring for operational state. Are all the BGP sessions that you expect to be up up? Or OSPF neighbors, or eVPN state, etc. If you don't measure all the important state, you can't know that things have failed. 

So you need to think about all the things that can fail, and make sure you have a way to monitor them all.

THen you need to be able to figure out how to turn that data into figuring out what's important.

## Alerts, Events, Alarms

I've gotten an alert, is it important? I didn't get an alert, are things okay? Something is failing, is that a problem?

One of the big issues with network health is that you get all kinds of signals from your data sources, but they aren't all important. If a link fails, is that bad? If you get errors from an optic, is that bad? A device reboots, is that bad? All of these questions require the context of the network and how the rest of the network is behaving. If you have only two routers of some kind, and one crashes, then that is bad. You can't afford to have the other one fail. But if you have 24 and the other 23 are working fine then everything is all right for now.


You also don't want to wake some up for every fault, because it might not be important and you can't afford to burn our your operations team on unimportant problems. It's  not okay to be waking up people when there isn't really a problem. It demotivates people and you miss important events


- Are the alarms I'm getting important and prioritized? Do I get woken up for something not important? When I get woken up, can I make sense of all the different information I get?
- When I get an alarm is it easy to action? Does it tell me where to look or is to to general?

## Describing what you expect
There are also a set of questions around if the network is as expected. If you just notice that the network is down or negatively effecting customers that's the symptoms. What if your network is sick, but you don't know it, and you want to prevent an outage before it happens? This requires
being able to describe what you expect your network to be. I guess this would be preventative health. Or like you want to catch the cancer before there are 

## you still don't get it all right
There are times in which all your systems are telling you things are running fine, but it turns out you aren't asking the right questions. Not exactly related, but in 2006 I was responsible for load balancers at Amazon, and I walked by  our VP of Infrastructure in the hall who asked me if we were doing fine for capacity. I said yes because all our indications were that it was fine. Then I started thinking about it more, and went home and started looking. I realized that we (I) had been incorrectly modeling capacity, and that we were already out of capacity under specific failure. You will get this wrong and you need to be able to ask and explore your network so that you can discover new things that you didn't realize you needed to care about.


## How to do this with Suzieq
Suzieq can't do all of these things yet, but it will be able to in the future. 

## Summary
1. You need to take responsiblity to know that the network is broken before customers. Understand that this is critical to doing a good job and don't be satisfied with what you currently have. This is more important than keeping up with whatever latest feature from your network vendor.
1. Healthy is more than just you aren't currently affecting customers. Do you have failures you don't know about that you could prevent from turning into customer problems?
1. Is your network behaving as expected?
1. Make your network as easy to measure as possible
1. Monitor all the things that can fail
1. Figure out how to aggregate and corelate alarms
1. Keep innovating to make your ability to know if you network is healthy or unhealthy better.

## Suzieq
Try out [Suzieq](https://www.stardustsystems.net/suzieq/), our open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.

## Conversation
1. Do you have a good way of describing if your network is healthy or not?
2. 