---
layout: post
comments: true
author: Justin Pietsch
title: Is my Network Healthy?
excerpt: 
description: 
---
Do you ever ask "Is my network healthy?" This is a critical question, but a hard and confusing one. It's also one that I think most people don't ask explicitly. What does it even mean that the network is healthy? And if I figure that out, how do I answer that question? I probably don't have the systems to answer that question well. Is it my responsibility? We have to figure out what it means to know that your network is healthy. What are we trying to understand when we ask that question? 

Sometimes it's when a customer calls up and says "My application isn't working, I've checked everything, I think the problem is the network." Sometimes it's when your CIO asks you "Is the network working correctly?" or worse "How many times this month has the network been unhealthy?"

Healthy doesn't just mean that there are no outages or no current customer complaints, though those are very important. I think it includes "Is the network trustworthy to do the things that we need to do it in?" That might be "I need to make a change, is the network healthy enough to do this change?" Or it might be "I need to add capacity, can we do that safely?" or upgrade the OS on all devices, or add a new application.

If you think it's your responsibility to know if your network is healthy or not then you have to think deeply about the questions you care about and how you can answer those. 

There are times in which all your systems are telling you things are running fine, but it turns out you aren't asking the right questions. Not exactly related, but in 2006 I was responsible for load balancers at Amazon, and I met our VP of Infrastructure in the hall and he asked me if we were doing fine for capacity. I said yes because all our indications were that it was correct. Then I started thinking about it more, and went home and started looking. I realized that we had been incorrectly modeling capacity, and that we were already out of capacity. In other words, you will get this wrong and you need to be able to ask and explore your network so that you can discover new things that you didn't realize you needed to care about.


Your network design is also critical to how you can answer and deal with this


### some definition of Healthy Network
Maybe it's easier to define a 

### Network Health Questions
* Is my network healthy?
* Is my network unhealthy?
* Is my network behaving as expected?


## Getting Started
I'd guess most networks don't really think about if their network is healthy. They start instead by thinking about what they can monitor and how to know that things are in Fault. But of course just because something is in Fault, doesn't mean that's a problem for the network. So can I define events as things that happen, like a router goes down or I get a weird syslog message. We have 


In truth it's very hard to define. We'll define it loosely as the ability for applications to be able to do what they want to do without the network causing trouble. That's probably good. The problem is that it is very hard to measure. How do you measure from all applications, especially if you are something like EC2 and customers might be doing anything. (In truth, Amazon has a lot of random things going on, it's been impossible to know what all internal traffic will do.)

## All my data sources

I know that I need a lot of data to know if my network is healthy or not right now. 

There are a lot of data sources in the network, and some of them are much harder to make useful than others. syslog might be critical, but super hard to make useful. I have experienced several very bad outages in which there was a message in syslog to notify us, but we'd never seen it before so we didn't know to look for it. (I wish software engineers and NOS vendors were oncall for their own products, I think things like that would happen less.)

## Alerts, Events, Alarms

One of the big issues with network health is that you get all kinds of signals from your data sources, but they aren't all important. If a link fails, is that bad? If you get errors from an optic, is that bad? A device reboots, is that bad? All of these questions require the context of the network and how the rest of the network is behaving. If you have only two routers of some kind, and one crashes, then that is bad. You can't afford to have the other one fail. But if you have 24 and the other 23 are working fine then everything is all right for now.


you also don't want to wake some up for every fault, because it might not be important and you can't afford to burn our your operations team on unimportant problems.

it's also not okay to be waking up people when there isn't really a problem. It demotivates people and you miss important events

I've gotten an alert, is it important?
I didn't get an alert, are things okay?

the problem with network alerts

- Are the alarms I'm getting important and prioritized? Do I get woken up for something not important? When I get woken up, can I make sense of all the different information I get?
- When I get an alarm is it easy to action? Does it tell me where to look or is to to general?

## Describing what you expect
There are also a set of questions around if the network is as expected. If you just notice that the network is down or negatively effecting customers that's the symptoms. What if your network is sick, but you don't know it, and you want to prevent an outage before it happens? This requires
being able to describe what you expect your network to be. I guess this would be preventative health. Or like you want to catch the cancer before there are 


## How to do this with Suzieq
Suzieq can't do all of these things yet, but it will be able to in the future. I'm not sure what we should be talking about.

## Suzieq
Try out [Suzieq](https://www.stardustsystems.net/suzieq/), our open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.