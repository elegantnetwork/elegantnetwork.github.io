---
layout: post
comments: true
author: Justin Pietsch
title: What I've learned about scaling OSPF in Datacenters
excerpt: One of the things I've learned recently is that OSPF can't be used for Clos leaf-spine networks. I've heard that it can't keep up with the scale of the modern network. ... This is a shock to me since we built large OSPF Clos networks a decade ago.
description: The wisdom is that BGP is the only Datacenter protocol, but is it? Do we know?
---
I worked at Amazon for 17 years as a network engineer. Now I'm out of Amazon and looking at the industry, I'm learning new things. **One of the things I've learned recently is that OSPF shouldn't be used for Clos leaf-spine networks because of scale.** I've heard that it can't keep up with the size and scale of the modern datacenter networks. [Juniper IP Fabric Underlay Network Design and Implementation](https://www.juniper.net/documentation/en_US/release-independent/solutions/topics/task/configuration/ip-fabric-underlay-cloud-dc-configuring.html) is an example of the recommendation.  <https://tools.ietf.org/html/rfc7938> is what as used as the reference for using eBGP to be the routing protocol inside of large Clos networks. 

**This is a shock to me** since we built large OSPF Clos networks a decade ago. We built networks this way with hundreds of routers (even thousands) speaking OSPF. I'm not saying it is easy or that we didn't have to understand OSPF well; If you don't design it well, you get a flooding disaster. But if you do set it up right, it works really well. It converges quickly and doesn't collapse under failure. The key thing is to be sure you understand how flooding works and you focus on areas and summarization. There are different things you have to be aware of when you are designing a Clos network with BGP vs OSPF. With BGP you have to be figure out how to mitigate path hunting. With OSPF need to work on how to avoid congestion collapse.

There is also work on building new protocols for these large Clos or Clos-like networks, such as <https://datatracker.ietf.org/wg/rift/documents/>. I don't understand why OSPF or ISIS isn't good enough so I don't understand why these are necessary. There [are some good ideas in RIFT](https://pc.nanog.org/static/published/meetings/NANOG74/1763/20181003_Martin_Routing_In_Dense_v1.pdf), I'm just haven't seen a reason to use it.

When we did this we were using CPUs from 2003 and we were using a lesser known OSPF stack. It's OSPF implementation was okay, but we were strongly warned away from using their BGP implementation by the developers. **We had to understand and mitigate risk**. We had to test it as well as we could and be very careful in our first deployments. The stack had never been battle tested like this before. We still ran into some scary performance issues but got to work with great software engineers on the protocol stack to work through problems and figure out our scale.

Others agree with me, <https://www.youtube.com/watch?v=Qmvg2mnbcPg>, even though they are mostly focusing on smaller networks than I was working on. But either way, OSPF can be made to work just fine on very large (or small) Clos networks.

As an example of the scale, this is a generic 24 port 3-tier Clos. It is possible to make OSPF work on this. This is smaller than what we were working on in 2012.
![24 port 3-tier clos](/assets/images/24port-3tier-clos-resize.png)

To understand and then mitigate the risks an unknown OSPF stack, at scale, is why I started building a configuration management system and started [simulating our network design](https://elegantnetwork.github.io/posts/Network-Validation-with-Vagrant/). We needed to understand how any OSPF protocol stack would work. Could we make the protocol scale at all? We needed to see what would actually happen with a real OSPF implementation. **I cannot stress enough how important this simulation was.** We did not understand the problem of flooding and we did not understand the solution until we had tried it out in simulation.

Before that, though, we spent a lot of time on white boards arguing about topology and OSPF. **I think whiteboards are the most important tool for network design** currently available, which makes me sad. I wish that wasn't true, I want much better tools. I can't even tell you the number of disasters averted by 2-3 great network engineers arguing over a whiteboard. This works really well, but there are important reasons it's not good enough. One is that for many of these discussions, it's extremely hard to get data to make the decisions. In this case, a comparison of convergence time and failure detection in a 3-tier Clos with eBGP, iBGP, and OSPF should be required, but we didn't have that capability. 

In a dense mesh-like network such as a Clos network, the concerns around OSPF swirled around four main issues: the effect of flooding, the size of the link state database, the speed of SPF calculations, and the ability for OSPF to carry a large number of prefixes. We were concerned about OSPF scaling w.r.t. its flooding and it's the flooding that is expensive with a Link State Database protocol, not the Shortest Path First (SPF) calculations. Which means you have to be very careful about what is flooded, what are the areas, and how things are summarized. I didn't understand this until I had a simulation and I could try things out with areas or without, and the effect was dramatic. I think it's often true that there is a part of a design that you don't understand, which is one of the big reasons to do simulation. We didn't have the ability to lab up a network as big as we were going to be building, so the simulation was critical. I can't even imagine what we would have done without it. Based on the simulation, we created targeted tests for real hardware to understand how it would deal with the scale. 

The point of this article is not that OSPF is better than BGP, but that it does work in very large size Clos networks. I'm frustrated that so much of the industry went a direction without great data or analysis, just following one set of opinions and design choices. **I wish there were easy ways for people to be able to design, simulate and test different designs** like this so that they could decide for themselves the tradeoffs they'd like to make, rather than have to rely on some type of industry hype. There needs to be a better way to be able to describe topologies and designs so that it's really easy to try out different ideas.

If you have other considerations, such as you want to run IPv6, or EVPN, a single instance of eBGP such as the one popularized by the open source routing suite, FRR, maybe the simplest or elegant. But with proper design, scale is not the reason to use BGP over OSPF. If you are running a small Clos network and don't have IPv6 or EVPN, OSPF is more straightforward and generally faster, as was the common practice before the advent of BGP in the data center.

No matter what, you need to understand the choices that you are making and understand the tradeoffs. This is hard to do, and especially hard in networking because we lack design patterns and good tools to help us understand the implications of our choices. 

If anybody knows about tools or design patterns or even somebody who's actually compared OSPF vs BGP in Clos networks with data, I'd love to hear about it.