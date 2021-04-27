---
layout: post
comments: true
author: Justin Pietsch
title: Who decides if the network is unhealthy?
excerpt: It's very important that the networking team that owns the design, build, and operations of the network is the responsible to know that the network it is operating is not working before it's customers do. This approach to networking will drive a network that is better monitored, probably better designed, and almost assuredly better for it's customers.
description: Do you rely on your customers to tell you our network is not working or do you take responsibility for network health? The networking team that owns the design, build, and operations of the network is the responsible to know that the network it is operating is not working before it's customers do.
---
When I started in networking at Amazon in the fall of 2002, when we were paged for a problem it was because one of the users/services on the network was having problems and they thought it was the network. We never knew ourselves that the network was broken until somebody told us it was. Maybe it was the website was down, or a Fulfillment Center was down, or somebody was having some weird kind of software problem they didn't understand. We did not have good tools to identify for ourselves that the network was misbehaving before our (internal) customers told us.

We had some monitoring if our devices went down (IIRC it didn't cut us tickets) but we di have good alerting for more than that. I would guess that most networks rely on their users/customers to tell them if the network is not working. . Even before AWS came along, it wasn't really viable for Amazon.com's networking team to not know it was having problems until it's customers told it, but this is really hard to do well. Some of it was we just didn't have the software necessary to do the right monitoring, and in the early 2000s, Amazon networking was small and didn't have a lot of budget, so we couldn't build our own. That changed overtime, and for a long time it's been true that AWS Networking expects to find problems before it's internal users or external customers. It continually pursues that and takes it very seriously.

**It's very important that the networking team that owns the design, build, and operations of the network is responsible to know that the network it is operating is not working before it's customers do.** This approach to operating your network will drive a network that is better monitored, better designed, and  assuredly better for it's customers. Taking a more relaxed approach implies that you are not taking your network as seriously as it's importance in the business or community that you are supporting.

Making this question primary will drive you to get better software to monitor your network. 

## Why does it matter who decides?

I have lived in a world in which the networking team didn't have the ability to know when it was likely that the network was broken, and I have lived in a world in which the networking team usually knew when it was broken. When you know before your customers, you can move faster in fixing the network. Maybe even more importantly you gain trust of your customers because they can believe you when you say that it isn't the network that is broken. You gain control of your destiny. If you don't think you own knowing when your network is available or not you don't feel like you own making your network available or not.

As important as the network is to everything in business and society today, it's not okay that the networking team doesn't know when it's broken. Every network needs to move that direction. Networks are just too important not to have the right visibility into understanding the network better than customers. If you are unwilling for the majority of outages to be caught by network users, you have to think differently about networking, monitoring, and alerting, as well as network design.

We want to raise the level of expectations of what a well run network are. This is just part of recognizing how important networks are and the responsibility because of their importance. To do it well means you need to continually work to figure out how to operate the network better. Detecting problems can be very difficult, and thinking about that might lead you to make all kinds of different decisions than if you assume it's somebody else's responsibility to say the network is down.

### What are the implications of taking this responsibility

It's very hard to answer the question for [Network Health](https://elegantnetwork.github.io/posts/network-health/) definitively: "Is the network working correctly?" First you have to start with accepting that you want to find the problems before your users/customers. If it's your responsibility to know before customers always (ok, not always, but that's the goal) then you have to start thinking about how you It possibly requires you to think about your network design. It most likely will change the way you think about operations. Hopefully it will drive you towards innovating in your network. It doesn't matter if these are hard to do, they are worth pursuing.

It's not a goal, **it's a way of life, a never ending pursuit**. You will never get perfect so you have to keep trying and keep innovating. Also, it will look different at different companies with different networks. For instance, if you have a fairly small and simple network, it might be really easy to know what constitute an outage. You might even have somebody just always looking at syslog and deciding if there is something wrong or not. I don't think this is a sustainable network, but it might be for some networks and some people

Accepting that it is your responsibility will change your monitoring, management, operations, and possibly even your network design.

## What does this mean for your network design

It will likely drive you towards simplicity. Owning [Network Health](https://elegantnetwork.github.io/posts/network-health/) is necessary to always know things are broken before your customers. If you must know if your network is healthy or unhealthy, then it might drive you to think about how to make your network easier to measure, and also have more obvious failures, and less grey failures.

This is much like having the responsibility for testing in software. If it's my responsibility to both write the software and show that it's working, I will spend more time and attention figuring out how to write software that is easier to test, easier to debug, and easier to monitor. The same needs to be true for networking. In networking it's often true that the people designing the network also run the network, but not always. But giving up the responsibility of finding the bugs also means you won't think as hard about how to find those issue.

One of the things that I don't understand about networking is that outages don't seem to drive to a simpler network. Outages seem to be blamed on the vendor, and we all update our software eventually (while cursing at the vendor) but I've rarely seen engineers think through the complexity of their solution and the likelihood that the complexity will bring mistakes in operating and bugs in the software on the devices. I wonder if because the health of the network is hard to measure, and so not measured well, it's very hard to track which things make availability better and which do not. And because it's not tracked over time there is less pressure to get better at it over time.

I think better software in monitoring, management, alert aggregation, etc. is the key, but it's not the only thing. It does take a better mindset change. Obviously it requires that you have good ways of measuring Network Health. But don't use your inability to measure the health as an excuse to not feel ownership for being able to answer if your network is working well.

If your management team is reluctant to invest in better ways to measure your network health, don't give up. Figure out ways to tell the story about why you need to be able to answer this question, to take control of your destiny, and how you need better ways of monitoring your network health.

## Summary

1. You need to take responsibility to know that the network is broken before customers. Understand that this is critical to doing a good job and don't be satisfied with what you currently have. This is more important than keeping up with whatever latest feature from your network vendor.
1. Own your destiny. 
1. Think through what it will take to take the responsibility of knowing when the network is working or not.

## Suzieq

Try out [Suzieq](https://www.stardustsystems.net/suzieq/), our open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.