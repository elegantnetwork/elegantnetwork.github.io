---
layout: post
comments: true
author: Justin Pietsch
title: Who decides if the network is unhealthy?
excerpt: It's very important that the networking team that owns the design, build, and operations of the network is the responsible to know that the network it is operating is not working before it's customers do. This approach to networking will drive a network that is better monitored, probably better designed, and almost assuredly better for it's customers.
description: Do you rely on your customers to tell you our network is not working or do you take responsibility for network health?
---
It's very important that the networking team that owns the design, build, and operations of the network is responsible to know that the network it is operating is not working before it's customers do. This approach to networking will drive a network that is better monitored, probably better designed, and almost assuredly better for it's customers. Taking a more relaxed approach implies that you are not taking your network as seriously as it's importance in the business or community that you are supporting.

When I started in networking at Amazon in fall of 2002, when we were paged for a problem it was because one of the users/services on the network was having problems and they thought it was the network. We never knew ourselves that the network was broken until somebody told us it was. Maybe it was the website was down, or a Fulfillment Center was down, or somebody was having some weird kind of software problem they didn't understand. We did not have good tools to identify for ourselves that the network was misbehaving before our (internal) customers told us.

We had some monitoring if our devices went down (IIRC it didn't cut us tickets) but we didn't have good alerting for more than that. I would guess that most networks rely on their users/customers to tell them if the network is not working. . Even before AWS came along, it wasn't really viable for Amazon.com's networking team to not know it was having problems until it's customers told it, but this is really hard to do well. Some of it was we just didn't have the software necessary to do the right monitoring, and in the early 2000s, Amazon networking was small and didn't have a lot of budget, so we couldn't build our own. That changed overtime, and for a long time it's been true that AWS Networking expects to find problems before it's internal users or external customers. It continually pursues that and takes it very seriously.

## Why does it matter who decides?

I have lived in a world in which the networking team didn't have the ability to know when it was likely that it was the thing broken, and I have lived in a world in which the networking team usually knew when it was broken. As important as the network is to everything in business and society today, it's not okay that the networking team doesn't know when it's broken. Every network needs to move that direction. Networks are just too important not to have the right visibility into understanding the network better than customers. If you are unwilling for the majority of outages to be caught by network users, you have to think differently about networking, monitoring, and alerting, as well as network design.

We want to raise the level of expectations of what a well run network are. This is just part of recognizing how important networks are and the responsibility because of their importance. To do it well means you need to continually work to figure out how to operate the network better. Detecting problems can be very difficult, and thinking about that might lead you to make all kinds of different decisions than if you assume it's somebody else's responsibility to say the network is down.

### What are the implications of taking this responsibility
It's very hard to answer the question definitively: "Is the network working correctly?" First you have to start with accepting that you want to find the problems before your users/customers. If it's your responsibility to know before customers always (ok, not always, but that's the goal) then you have to start thinking about how you It possibly requires you to think about your network design. It most likely will change the way you think about operations. Hopefully it will drive you towards innovating in your network.


It's not a goal, **it's a way of life, or a pursuit**. You will never get perfect so you have to keep trying and keep innovating. Also, it will look different at different companies with different networks. For instance, if you have a fairly small and simple network, it might be really easy to know what constitute an outage. You might even have somebody just always looking at syslog and deciding if there is something wrong or not. I don't think this is a sustainable network, but it might be for some networks and some people

Accepting that it is your responsibility will change your monitoring, management, operations, and possibly even your network design.

It’s too hard to do the right thing for lots of companies. I’m not sure that there is a simple solution that you can buy that really gives you the confidence you need. You have to assemble lots of pieces and put them together. 
### What does it mean for the Network to be Unhealthy

You have to start by deciding what it means for a [network to be unhealthy](https://elegantnetwork.github.io/posts/network-health/). Does it mean that a device is down or an interface or a BGP session? No, probably not. It means it's getting in the way of what the users of the network are trying to do. For some measure of unhealthy, it's much easier to monitor from the application point of view. But on the other hand, if there are many applications, how do you measure all of that?

### Monitoring and Data
What would be the minimum
- Fault monitoring 
- Performance
- Event Monitoring
- Active Monitoring
- Capacity management
- Good tickets prioritization
- Have to have the right Attitude. Is it more important to know right away or to not wake people up. But you also don’t want 
- A way to prioritize
- Be able to check more than just standard metrics
	- Like are all the necessary bgp peers up. 
	- Do you have the right routes in the right places. Canary checking. 

### active and Passive Monitoring


### Challenges

There are many challenges. I don't think any network hardware vendor takes that kind of monitoring seriously. Though it's been a long time, I used to ask vendors if they had a counter for every possible packet drop. I've not gotten a good answer. At one place, the hardware people said "Of Course!" and the software people said: "". It's actually hard to present those counters in a useful way and not make mistakes like over-counting. However, it doesn't matter if it's hard, we have to convince everybody that it's required.

### grey failures are especially hard

I think pingmesh is great way to do this, but it is really really hard to do well.


I wrote about the network being in the way. Not knowing that the network has failed until a customer tells you means that you are in the way




## What does this mean for your network design


drive towards simplicity.


This is much like having the responsiblity for testing in software. If it's my responsbility to both write the software and show that it's working, I will spend more time and attention figuring out how to write software that is easier to test, easier to debug, and easier to monitor. The same needs to be true for networking. In networking it's often true that the people designing the network also run the network, but not always. But giving up the responsibility of finding the bugs also means you won't think as hard about how to find those issue.s


I think better software is the key, but it's not the only thing.

network validation

## Suzieq
Try out [Suzieq](https://www.stardustsystems.net/suzieq/), our open source, multivendor tool for network observability and understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.