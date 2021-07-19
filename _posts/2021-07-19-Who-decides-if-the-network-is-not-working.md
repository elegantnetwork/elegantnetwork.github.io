---
layout: post
comments: true
author: Justin Pietsch
title: Who decides if the network is unhealthy?
excerpt: It's very important that the networking team that owns the design, build, and operations of the network is responsible to know that the network it is operating is not working before it's users do.
description: The networking team that owns the design, build, and operations of the network is responsible to know that the network it is operating or not working before it's users do.
---
I started in networking at Amazon in the fall of 2002. When we were paged for a problem it was because one of the users/services on the network was having problems and they thought it was the network. We never knew ourselves that the network was broken until somebody told us it was. Maybe the website was down, or a warehouse was down, or somebody was having some weird kind of software problem they didn't understand. We did not have good tools to identify for ourselves that the network was misbehaving before our (internal) users told us.

We had some monitoring if our devices went down (IIRC it didn't cut us tickets) but we did not have good alerting for more than that. I would guess that most networks rely on their users/users to tell them if the network is not working. Even before AWS came along, it wasn't viable for Amazon.com's networking team to not know it was having problems until it's users told it, but this is really hard to do well. Some of it was we just didn't have the software necessary to do the right monitoring, and in the early 2000s, Amazon networking was small and didn't have a lot of budget, so we couldn't build our own. That changed overtime, and for a long time it's been true that AWS Networking expects to find problems before it's internal users or external users. It continually pursues that and takes it very seriously.

**It's very important that the networking team that owns the design, build, and operations of the network is responsible to know that the network it is operating is not working before it's users do.** This approach to operating your network will drive a network that is better monitored, better designed, and assuredly better for it's users. Taking a more relaxed approach implies that you are not taking your network as seriously as it's importance in the business or community that you are supporting.

Making this question primary will drive you to get better software to monitor your network.

## Why does it matter who decides?

I have lived in a world in which the networking team didn't have the ability to know when it was likely that the network was broken, and I have lived in a world in which the networking team usually knew when it was broken. When you know before your users, **you can move faster in fixing the network.** Maybe even more importantly **you gain trust of your users** because they can believe you when you say that it isn't the network that is broken. You gain control of your destiny. If you don't think you own knowing when your network is available or not you don't feel like you own making your network available or not.

If your goal is to know that the network is impacted before customers, then you will measure how well you do that. As you measure, you will then focus on improving it. 

As important as the network is to everything in business and society today, it's not okay that the networking team doesn't know when it's broken. Every network needs to move that direction. Networks are just too important not to have the right visibility into understanding the network better than users. If you are unwilling for the majority of outages to be caught by network users, you have to think differently about networking, monitoring, and alerting, as well as network design.

We want to raise the level of expectations of what a well run network are. This is just part of recognizing how important networks are and the responsibility because of their importance. To do it well means you need to continually work to figure out how to operate the network better. Detecting problems can be very difficult, and thinking about that might lead you to make all kinds of different decisions than if you assume it's somebody else's responsibility to say the network is down.

### What are the implications of taking this responsibility

It's very hard to answer the question for [Network Health](https://elegantnetwork.github.io/posts/network-health/) definitively: "Is the network working correctly?" First you have to start with accepting that you want to find the problems before your users/users. If it's your responsibility to know before users always (ok, not always, but that's the goal) then you have to start thinking about how you It possibly requires you to think about your network design. It most likely will change the way you think about operations. Hopefully it will drive you towards innovating in your network. It doesn't matter if these are hard to do, they are worth pursuing.

It's not a goal, **it's a way of life, a never ending pursuit**. You will never get perfect so you have to keep trying and keep innovating. Also, it will look different at different companies with different networks. For instance, if you have a fairly small and simple network, it might be really easy to know what constitute an outage. You might even have somebody just always looking at syslog and deciding if there is something wrong or not. I don't think this is a sustainable network, but it might be for some networks and some people

Accepting that it is your responsibility will change your monitoring, management, operations, and possibly even your network design.

## How to get started

**What I most want from this post is that network operators and engineers have the mindset that they should own discovering and deciding that the network is broken**. However, after that comes mechanisms, software, and processes to actually do this. I think better software in monitoring, management, alert aggregation, etc. is key, but it's not the only thing and not where to start. It does take a better mindset change. Obviously it requires that you have good ways of measuring Network Health. But don't use your inability to measure the health as an excuse to not feel ownership for being able to answer if your network is working well.

First off, I hope you have **much better** monitoring than we had in 2002. You should have a alarm/ticketing system that knows when every device (and link) goes down and some process to decide when these are escalated to operators. You have to start with a system measuring your network health and you will need to continue to make it better.

The next piece might be to measure how many outages you have that were detected by automation in networking vs some other outside person or system. This is most likely a manual system, but probably isn't too onerous. What you measure and have goals on you will optimize and this is a goal worth optimizing towards. It might make changes in your network or software systems required, and that is good.

You should have some kind of [process for evaluating outages](https://wa.aws.amazon.com/wat.concept.coe.en.html). In this process you should spend time on if the outage was detected automatically or not and what can be done to make the detection better. 

You can't have perfection, but even getting to 50% is good, and 80/20 is great. Keep getting better, an outage at a time. If your management team is reluctant to invest in better ways to measure your network health, don't give up. Figure out ways to tell the story about why you need to be able to answer this question, to take control of your destiny, and how you need better ways of monitoring your network health.
## What does this mean for your network design

**It will likely drive you towards simplicity.** Owning [Network Health](https://elegantnetwork.github.io/posts/network-health/) is necessary to always know things are broken before your network users. If you must know if your network is healthy or unhealthy, then it might drive you to think about how to make your network easier to measure, and also have more obvious failures, and less grey failures.

This is much like having the responsibility for testing in software. If it's my responsibility to both write the software and show that it's working, I will spend more time and attention figuring out how to write software that is easier to test, easier to debug, and easier to monitor. The same needs to be true for networking. In networking it's often true that the people designing the network also run the network, but not always. But giving up the responsibility of finding the bugs also means you won't think as hard about how to find those issue.

## Summary

We want to keep raising the bar of what is expected of networking.

1. You need to take responsibility to know that the network is broken before users. Understand that this is critical to doing a good job and don't be satisfied with what you currently have. This is more important than keeping up with whatever latest feature from your network vendor.
1. Own your destiny.
1. Think through what it will take to take the responsibility of knowing when the network is working or not.

## Suzieq

Try out [Suzieq](https://www.stardustsystems.net/suzieq/), the open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network. Suzieq collects operational state in your network and lets you find, validate, and explore your network.
