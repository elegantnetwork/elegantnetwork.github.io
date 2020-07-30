---
layout: post
comments: true
author: Justin Pietsch
title: Lessons for network engineers from load balancers and multicast
excerpt: In general I dislike load balancers and IP multicast ... But they've been very important for my career.
description: Supporting load balancing and multicast helps understand the applications and services on the network.
---

In general I dislike load balancers and IP multicast: I'm a network engineer. They are very complicated, have a 
large amount of state which is often in the middle of the network, and they are hard to understand and debug. 
They lead to a lot of operational issues and outages. They have directly lead to a lot of pages 
in my life and a lot of lost sleep. They are hard to capacity plan, fail in un-intuitive ways, etc. 

I could go on.

I'm not the only one. There is a deep hatred inside of much of Amazon/AWS for hardware load balancers. I'm not saying it's  justified, but it exists. I think the load balancing is harder than people want to admit.

In my ideal datacenter network, I just want to deal with unicast IPv4, OSPF and BGP. It's hard enough just getting the basics running. I want a simple, uncomplex network. Components that add more state into the network make the network fragile, they make the whole system less stable. They’ve lead to an alarming amount of outages in my experience. I don’t think any other technologies or protocols have caused as much trouble in the datacenter networks that I've had responsibility for. [This outage in 2004, a month after a bunch of load balancer outages](https://www.cnet.com/news/amazon-com-hit-with-outages/) was worse, but it wasn't the network. Databases, not my fault.

But they've been very important for my career.

From an application point of view load balancers are extremely important, or at least load balancing is. There is a need for abstracting services apart from each other, doing load balancing. and health checking. Multicast on the other hand, nope. Too hard. Not worth the effort. For a long time I thought it was cool technology that just needed to be worked out, but I haven't thought that since 2005.

When I started at Amazon in September of 2002, we had issues because our load balancers were our L3 gateway, and so sometimes multicast traffic through the load balancer got dropped. I don’t know the Amazon website ever worked at that point in time. In the first several months I was at Amazon in late 2002 or early 2003 (I can't remember which month, it was a blur), we came upon a period that every night at the same time we would have a network outage and we did not know what was going on. I can't remember how we narrowed things down to specific devices, but at some point we got Vendor support to log into our devices and do some debugging*. The engineer (I think it was m@vendor) reported that we were dropping packets on some specific ASIC (Pinnacle) in our line cards. We’d get drops on the device and not have any counters reporting drops on anything that we could access. Silent drops are the worst, but a topic for another time. What can you do about that? I learned that you have to pay attention to vendor architecture and it's worth knowing the day-in-the-life-of-a-packet. This was a very important lesson for me. And we had some serious conversations about counters!

Now what traffic did we have that caused packet drops? We had a content push at this time of night, which was multicast, every night, and it went to many different switch ports. I can't remember what the band-aid was that we used to mitigate the problem the night that we finally understood what was going on. I also remember that the night we figured it out we got a call from our VP demanding that we figure it out while there was oonce oonce music in the background of his call. Thanks, yes, we'll get on that!

Over time we spent a lot of time with load balancer vendors. We ran into scaling issues, feature issues, and we needed better ideas on how to manage and operate what became so many LBs.  We thought 10 pairs of load balancers was a lot, then 100 pairs, etc. That is not a lot, compared to what Amazon has now. But LBs are hard to manage and it's hard to figure out how to best manage VIPs for internal usage. It took a long time to figure out a good pattern, and then build software to manage that. One of the hard part about load balancers when you have hundreds, then thousands, then 10s of thousands of services that need load balancing is that all those applications change a lot and the teams that manage those need to make changes to their load balancing a lot. That provisioning and changes need software.

We once had a bug with our load balancer in which some of the time on the heaviest VIPs we would get small amounts of connection drops that would effect their 99.9th percentile latency. 99.9th percentile is probably the most important metric to amazon: it gives an understanding of your outlier performance, but isn't too biased by a few bad interactions. We couldn’t get the Vendor's software team to take the issue seriously. Finally our sales engineer got me device code. In the middle of the night I woke up and realized what was wrong. And then figured out a solution. We already didn’t trust those LBs, but that was the last straw for me.

We learned a lot about the technology, spent a lot of time working with engineers at vendors. Influenced the products. I learned a lot about network hardware. I got to meet some fantastic engineers at network hardware vendors. The biggest lessons I learned are that running out of capacity is the worst thing you can do in networking. There are more resources that limit capacity than you realized at first. Your job as an engineer is to find what those are. We used least connections load balancing, and one of our load balancers looked linearly through the servers to find which had the least connections. We thought that we had n*10s of thousands of connections per second, but that was drastically reduced because the highest throughput service had hundreds of servers in it. We didn't know to test for that, and we had a set of huge outages.

But those were not the most important lessons. 

The reason multicast was interesting to Amazon in late 2002 and on was that Amazon started using Tibco Rendezvous** for messaging for almost all traffic between services. A client would send a request out to a multicast group, which would get picked up by a queuing service, which would then send it to a service host. This is some pretty cool infrastructure magic. Clients, queues, and servers can all be nicely separate from each other. But then we had so many disasters, it became clear that this was not the path to victory.

In 2004 and 2005, I believe we had the most S,Gs in any datacenter network in the world, primarily because the hardware available couldn't handle anymore. I don't remember exactly, but I believe we had 16K S,Gs, then a code change, and then up to 20K. At least it was something around that. If you don't know what S,Gs are, they are source, group pairs, which is the state that multicast routers have to keep to know where to send traffic. 

We talked to our vendor, who had been working on PIM-BIDIR, a new version of the PIM protocol that kept state based on just groups. The primary customer for PIM-BIDIR was financial institutions who were more conservative than us. I remember in the summer or fall or 2004 at the vendor's talking about when we would do this. We said we couldn't do it in time for Christmas, but we'd want to do it in the first quarter of 2005. They had assumed more like 18-24 months. That was a great experience, working with high quality engineers who really cared about getting this working. We did have bugs and it was hard, but it was a great success. Even after this, multicast was a huge problem for us.

I'm not going to go through all the reasons that multicast was a terrible idea or all the  outages. Oh, except one more. We were mutlicasting all log messages from servers to a fleet of processing hosts. The cool thing is that the clients didn't need to know anything about the processing, and the processing hosts behind the scenes can change how they broke up processing. Again, very cool magic infrastructure. One night I was oncall, probably early 2005. Somebody had accidentally broken the webserver cache on an Amazon website. There was a service that returned image size, which got a greater than 90% hit rate on that cache. So the service got 10x the traffic it was expecting and started timing out requests. There was a log message for every miss and because logs were multicast, we were browning out interfaces all over the datacenter with multicast. Untying that mess took a lot of time and more network engineers than me.

I learned how to think about scaling networks and how critical it is to make things simpler. How to think about tradeoffs around magic abstractions and understandability. Infrastructure that is magic is often too good to be true, at least when you are scaling and growing very quickly. It requires deep introspection, understanding of what happens under failure, and some great monitoring.

But I think even this wasn't the most important lessons I learned.

Except I have been leaking the lessons throughout this blog-rant. I got to learn all about applications and I got to learn all about how Amazon.com worked. In some ways, the best way to learn is through outages and we had a lot of them in the early 2000s (I think one of the critical reasons AWS is so good is because of the leaning about distributed systems during this period). I also got to learn about the most important services and how they thought about scale.

In 2005 and especially in 2006 I owned load balancers, and at that time the biggest problem was understanding capacity. To understand capacity, you have to understand the traffic in aggregate. After some analysis I realized that the top 10 services (of thousands at that point) did about 50% of the traffic. This was a power law, which helped me understand that the focus has to be on those 10 services. You can probably guess what those services are, based on an eCommerce website. We had to spend time with those top services understanding how they scale. Just because they'd said 20% growth didn't mean we weren't still responsible when it turned out to be %100. Or when VPs said no more hardware load balancers, but the website demanded 2x, still our problem. Also, just because sales grow by 20% (or whatever the number is,) doesn't mean page hits or calls to search go up by only 20%.

I learned a lot about the Amazon architecture. I had the experience of fighting VPs; I won't say I learned how to do that well. There is so much interesting to learn from distributed systems and software engineering that can be translated into how to scale networks well. For instance, every time you can make a hard failure, rather than a partial or grey failure is a big win. Grey failures are extremely hard to detect. Exponential backoff is really a great thing (ok, maybe I already knew that one.)

I met so many interesting people and projects. LBs are how I met many of the people on the S3 team, and over time I got to help them out from time to time (doing non-LB work.) I still think S3 is the most amazing technology I’ve ever seen. I remember meeting the first five people on the Amazon unbox team, and Amazon Video is now over a thousand of people. I have some vague and fading knowledge about how eCommerce sites work and how hard it is to have a very large catalog.

I think networking is very interesting field; that’s why I’m still in it. But I’ve also really enjoyed learning about the technology that got built around me. And I got to learn how to think about applications, engineering, design, and architecture from more than just the network engineers around me.

 

*Anybody at Amazon these days hearing about a vendor logging into devices is probably throwing up right now. This has not happened for a long time.

**One of my two most hated products * vendor interactions. At one point they effectively told us it was our fault we had so much trouble because we ran both 100M and 1G Ethernet. Nope, the product sucked. And it was a bad idea even if the product was good software. I still feel outrage 17 years later.