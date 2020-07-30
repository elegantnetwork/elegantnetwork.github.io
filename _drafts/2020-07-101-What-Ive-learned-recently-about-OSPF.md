---
layout: post
comments: true
author: Justin Pietsch
title: What I've learned recently about OSPF in Datacenters
excerpt: One of the things I've learned recently is that OSPF can't be used for Clos leaf-spine networks. I've heard that it can't keep up with the scale of the modern network. ... This is a shock to me since we built large OSPF Clos networks a decade ago.
description: The wisdom is that BGP is the right Datacenter protocol, but is it? Do we know?
---
I worked at Amazon for 17 years as a network engineer. As you might expect, Amazon doesn't really keep up with what's going on in the industry, certainly the last decade as they've tried to build what they need. Amazon tries to do everything themselves. And we were too busy trying to make that all work to really pay attention to what anybody else was doing.

I'm out of Amazon and looking at the industry and I'm learning new things. **One of the things I've learned recently is that OSPF can't be used for Clos leaf-spine networks.** I've heard that it can't keep up with the scale of the modern network. <https://www.juniper.net/documentation/en_US/release-independent/solutions/topics/task/configuration/ip-fabric-underlay-cloud-dc-configuring.html> is an example of the recommendation.  <https://tools.ietf.org/html/rfc7938> is what as used as the reference for using eBGP to be the routing protocol inside of large Clos networks. I did know about this RFC, but I'm surprised how almost everybody seems to be using BGP in Clos networks.

**This is a shock to me** since we built large OSPF Clos networks a decade ago. We built Clos networks this way with hundreds of routers (even thousands) speaking OSPF. I'm not saying it wasn't tricky, it doesn't require understanding OSPF well and making sure you get it right. It does, and if you don't design it well, you get a flooding disaster. But if you do set it up right, it works really well. **It converges quickly and doesn't collapse under failure.**

There is also work on building new protocols for these large Clos or Clos-like networks, such as <https://datatracker.ietf.org/wg/rift/documents/>. I don't understand why OSPF or ISIS isn't good enough so I don't understand why these are necessary. Also, as far as I can tell by skimming through RIFT, it has some important limitations

When we did this we were using CPUs from 2003 and we were using a not well known OSPF stack that had never been abused to this degree. For reasons, we had to use this protocol stack. It's OSPF implementation was okay, but we were strongly warned away from using BGP by the developers. **We had to understand and mitigate risk**. We had to test it as well as we could and be very careful in our first deployments. We still ran into some scary performance issues but worked with great software engineers on the protocol stack to figure out and scale.

Others agree with me, <https://www.youtube.com/watch?v=Qmvg2mnbcPg>, even though they are mostly focusing on smaller networks than I was working on. But either way, OSPF can be made to work just fine on very large (or small) Clos networks.

As an example of the scale, this is a generic 24 port 3-tier Clos. It is possible to make OSPF work on this. Actually, this is smaller than what we were working on in 2012.
![24 port 3-tier clos](/assets/images/24port-3tier-clos.svg)

To understand and then mitigate is why I started building a configuration management system and started simulating (https://elegantnetwork.github.io/posts/Network-Validation-with-Vagrant/) our network. We needed to understand how any OSPF protocol stack would work. Could we make the protocol scale at all? We could, independent of the protocol stack. We did a lot of work on a whiteboard, but at some point we needed to see what would actually happen with a real OSPF implementation.

Before that, though, of course we spent a lot of time on white boards arguing about topology and OSPF. **I think whiteboards are the most important tool for network design**. I wish that wasn't true, I wish we had much better tools. I can't even tell you the number of disasters averted by 2-3 great network engineers arguing over a whiteboard. This works really well, but there are important reasons it's not good enough. One is that for many of these discussions, it's extremely hard to get data to make the decisions. In this case, a comparison of convergence time and failure detection in a 3-tier Clos with eBGP, iBGP, and OSPF should be required, but we didn't have that capability. 

**In a dense mesh-like network such as a Clos network, it's the flooding that is expensive** in a Link State Database, not the Shortest Path First (SPF) calculations. Which means you have to be very careful about what is flooded, what are the areas, and how things are summarized. I didn't understand this until I had a simulation and I could try things out with areas or without, and the effect was dramatic.

We didn't have the ability to lab up a network as big as we were going to be building, so the simulation was critical. I can't even imagine what we would have done without. Based on the simulation, we then could test a small amount of real hardware to understand how it would deal with the scale. 

In the end, I'm not saying that OSPF is better than BGP for a Clos network and I'm certainly not saying that BGP is better than OSPF. **I wish there were easy ways for people to be able to design, simulate and test different designs** like this so that they could decide for themselves the tradeoffs they'd like to make, rather than have to rely on some type of industry hype. I sure wish we had better ways to compare these different design choices and really understand their impact. There needs to be a better way to be able to describe topologies and designs so that it's really easy to try out different ideas.

Some large part of the problem is that most of the design patterns that are used in networking are from vendors and since there are not good ways to try out different patterns, those are just accepted. I think we need a better way of creating, testing, and sharing design patterns in networking. Most design patterns come directly from Equipment vendors who are both not necessarily incented to for the best overall design as well as they don't have experience actually operating networks.

No matter what, you need to understand the choices that you are making and understand the tradeoffs. This is hard to do, and especially hard in networking because we lack design patterns and good tools to help us understand the implications of our choices. 

If anybody knows about tools or design patterns or even somebody who's actually compared OSPF vs BGP in Clos networks with data, I'd love to hear about it.