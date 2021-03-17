---
layout: post
comments: true
author: Justin Pietsch
title: Who decided if the network is not working.
excerpt: 
description: Do you rely on your customers to tell you our network is not working or do you take responsibility for network health?
---

When I started in networking at Amazon in fall of 2002, most of the time when we were paged for a problem it was because one of the users/services on the network was having problems and they thought it was the network. Maybe it was the website was down, or a Fulfillment Center was down, or somebody was having some weird kind of software problem they didn't understand. We did not have good tools to identify for ourselves that the network was misbehaving before our (internal) customers told us.

We had some monitoring if our devices went down (though it might not have even cut us tickets, it was so long ago), but more than that we didn't have good alerting. I would guess that most networks rely on their users/customers to tell them if the network is not working. It's extremely hard to know all the ways a network can fail and then monitor for that. Even before AWS came along, it wasn't really viable for Amazon.com's networking team to not know it was having problems until it's customers told it, but this is really hard to do well. Some of it was we just didn't have the software necessary to do the right monitoring, and in the early 2000s, Amazon networking was small and didn't have a lot of budget, so we couldn't build our own. That changed overtime, and for a long time it's been true that AWS Networking expects to find problems before it's internal users or external customers. It continually pursues that and takes it very seriously.

I think every network needs to move that direction. Networks are just too important not to have the right visibility into understanding the network better than customers. If you are unwilling for the majority of outages to be caught by network users, you have to think differently about networking, monitoring, and alerting, as well as network design.

It's very hard to answer the question definitively: "Is the network working correctly?" First you have to start with accepting that you want to find the problems before your users/customers. 
There are many challenges. I don't think any network hardware vendor takes that kind of monitoring seriously. Though it's been a long time, I used to ask vendors if they had a counter for every possible packet drop. I've not gotten a good answer. At one place, the hardware people said "Of Course!" and the software people said: "". It's actually hard to present those counters in a useful way and not make mistakes like over-counting. However, it doesn't matter if it's hard, we have to convince everybody that it's required.

grey failures are especially hard

I think pingmesh is great way to do this, but it is really really hard to do well.


I wrote about the network being in the way. Not knowing that the network has failed until a customer tells you means that you are in the way


we want to raise the level of expectations of what a well run network are.

It’s too hard to do the right thing for lots of companies. I’m not sure that there is a sine solution that you can buy that really gives you the confidence you need. You have to assemble lots of pieces and put them together. 

What would be the minimum
- Fault monitoring 
- Performance
- Good tickets prioritization
- Have to have the right Attitude. Is it more important to know right away or to not wake people up. But you also don’t want 
- A way to prioritize
- Be able to check more than just standard metrics
	- . Like are all the necessary bgp peers up. 
	- Do you have the right routes in the right places. Canary checking. 

drive towards simplicity.

Why is this important

If it's your responsibility to know before customers always (ok, not always, but that's the goal,) then you have to start thinking about how you can possibly pull that off. It requires better ways of monitoring and observing your network. It might require active monitoring (pingmesh). It possibly requires you to think about your network design. It most likely will change the way you think about operations. Hopefully it will drive you towards innovating in your network.

I think better software is the key, but it's not the only thing.

network validation