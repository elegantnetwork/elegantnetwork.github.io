---
layout: post
comments: true
author: Justin Pietsch
title: Is my Network Healthy?
excerpt: 
description: 
---
Do you ever ask "Is my network healthy?" This is a critical question, but a hard and confusing one. What does it even mean that the network is healthy? And if I figure that out, how do I answer that question? I probably don't have the systems to answer that question well. Is it my responsibility?

I think it is your responsibility [to decide the network is not healthy before your customers do](https://elegantnetwork.github.io/posts/Who-decides-if-the-network-is-not-working/). Which means we have to figure out what it means to know that your network is healthy. What are we trying to understand when we ask that question? 

Sometimes it's when a customer calls up and says "My application isn't working, I've checked everything, I think the problem is the network." Sometimes it's when your CIO asks you "Is the network working correctly?" or worse "How many times this month has the network been unhealthy?"

I think there are also a set of questions around if the network is  as expected. 

I think healthy doesn't just mean that there are no outages or no current customer complaints, though those are very important. I think it includes "Is the network trustworthy to do the things that we need to do it in?" That might be "I need to make a change, is the network healthy enough to do this change?" Or it might be "I need to add capacity, can we do that safely?" or upgrade the OS on all devices, or add a new application.

If you think it's your responsibility to know if your network is healthy or not


There are times in which all your systems are telling you things are running fine, but it turns out you aren't asking the right questions. Not exactly related, but in 2006 I was responsible for load balancers at Amazon, and I met our VP of Infrastructure in the hall and he asked me if we were doing fine for capacity. I said yes because all our indications were that it was correct. Then I started thinking about it more, and went home and started looking. I realized that we had been incorrectly modeling capacity, and that we were already out of capacity. In other words, you will get this wrong and you need to be able to ask and explore your network so that you can discover new things that you didn't realize you needed to care about.


Your network design is also critical to how you can answer and deal with this


## All my data sources
there are a lot of data sources in the network, and some of them are much harder to make useful than others. syslog might be critical, but super hard to make useful. I have experienced several very bad outages in which there was a message in syslog to notify us, but we'd never seen it before so we didn't know to look for it. (I wish software engineers and NOS vendors were oncall for their own products, I think things like that would happen less.)

## Alerts, Events, Alarms

Ive 


it's also not okay to be waking up people when there isn't really a problem. It demotivates people and you miss important events

I've gotten an alert, is it important?
I didn't get an alert, are things okay?

the problem with network alerts

- Are the alarms I'm getting important and prioritized? Do I get woken up for something not important? When I get woken up, can I make sense of all the different information I get?
- When I get an alarm is it easy to action? Does it tell me where to look or is to to general?

## Describing what you expect