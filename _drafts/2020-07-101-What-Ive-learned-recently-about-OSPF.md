---
layout: post
comments: true
author: Justin Pietsch
title: What I've learned recently about OSPF in Datacenters
excerpt: 
description: The wisdom is that BGP is the right Datacenter protocol, but is it? Do we know?
---
I worked at Amazon for 17 years as a network engineer. As you might expect, Amazon doesn't really keep up with what's going on in the industry, certainly the last decade as they've tried to build what they need. Amazon tries to do everything themselves. And we were too busy trying to make that all work to really pay attention to what anybody else was doing.

So as I'm out of Amazon and looking at the industry I'm learning new things. One of the things I've learned is that OSPF can't be used for Clos leaf-spine networks. I've heard that it can't keep up with the scale of the modern network. https://tools.ietf.org/html/rfc7938.


This is a shock to me since this is exactly what we built a decade ago. And we built really large Clos networks this way with hundreds of routers (even thousands) speaking OSPF. I'm not saying it wasn't tricky, it doesn't require understanding OSPF well and making sure you get it right. It does, and if you don't design it well, you get a flooding disaster. But if you do set it up right, it works really well. It converges quickly and doesn't collapse under failure.

And just to be clear, when we did this we were using CPUs from 2003 and we were using a not well known OSPF stack that had never been abused to this degree. We had to mitigate risk anyway that we could. We did. We had to test it as well as we could and be very careful in our first deployments. We still ran into some scary performance issues but worked with great software engineers on the protocol stack to figure out and scale.

Others agree with me, https://www.youtube.com/watch?v=Qmvg2mnbcPg, even though they are mostly focusing on smaller networks than I was working on.

 This is why I started building a configuration management system and started simulating (https://elegantnetwork.github.io/posts/Network-Validation-with-Vagrant/) our network. We needed to understand how any OSPF protocol stack would work. Could we make the protocol scale at all? We could, independent of the protocol stack. We did a lot of work on a whiteboard, but at some point we needed to see what would actually happen with a real OSPF implementation.

By the way, in a dense mesh-like thing such as a Clos network, it's the flooding that is expensive, not the SPF. Which means you have to be very careful about what is flooded, and how things are summarized.

I'm not saying that OSPF is better than BGP for a Clos network and I'm certainly not saying that BGP is better than OSPF. I wish there were easy ways for people to be able to design, simulate and test different designs like this so that they could decide for themselves the tradeoffs they'd like to make, rather than have to rely on some type of industry hype.

I sure wish we had better ways to compare these different design choices and really understand their impact. There needs to be a better way to be able to describe topologies and designs so that it's really easy to try out different ideas.