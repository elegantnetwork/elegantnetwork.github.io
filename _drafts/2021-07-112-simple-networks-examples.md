---
layout: post
comments: true
author: Justin Pietsch
title: Examples of simplifying networking
excerpt:
description: What does it mean to simplify your network? Here are some examples and why they were important for Amazon.
---

You've probably heard people say that networks are too complex and need to be made simpler. I certainly believe that. I think people build too complicated networks and don't consider how hard it will be to operate those over time. There are many reasons: people believe vendor hype, or like shiny things, or the business asks for things that require complexity.

I think it might be helpful to go through examples I've experienced of simplifying networks. Often it does require a discussion with the people using the network. Remember, **engineering is the art of tradeoffs*. You must weight the complexity of operating all the features you've been asked for and ask if there is a better way of doing that. You will get different answers with different people, and different companies/cultures involved in the discussion, but the point is to do the analysis and understand the tradeoffs.


## L2/Vlans

When I started at Amazon in late 2002, we had two datacenters and they were filled with L2 and vlans. There were three main networks: website, dbnet, and corporate. Each host in the datacenter had two NICs and were attached to two of the three networks, depending on the purpose of the host. I wasn't around when these were made, I think the idea was to keep the database hosts on dbnet secure from the internet, they were on the dbnet and the corporate net, but not on the website. Website hosts were on website and dbnet but not corporate. I don't know how hosts were decided on which vlan they were on inside those networks.

In the early 2000s, Load Balancers were mostly L2 based, at least the ones that Amazon used. Which means that the LB needed to be the default gateway for devices that needed load balancing. IIRC, if one host needed to talk to a service that had hosts on the same vlan, then they couldn't talk successfully, so that service needed to be moved to another VLAN.

We also started using multicast the summer of 2002, and remember that the LB was the default gateway, so they got all the multicast.

As you can imagine, lots of outages and lots of operational nightmares. Hard to keep up and hard to scale. I think most people know to hate L2 and vlans and spanning tree these days, but that was less so in the early 2000s.

Maybe you've heard of Amazon's famous 2-pizza teams. The concept is a little overblown, things didn't really turn out that way, but it's true that the culture was made so that teams could (and did) spin up as many different services that they needed to build the things that they needed for customers. This all lead to an explosion of software, interacting with other software in ways that nobody truly understood or understands.

If we had had the ability to think ahead we would have seen that this couldn't possibly work. Instead I think enough of us were offended by L2 and the mess that we just argued to make it simpler. How could we decouple this knot of confusion? The first step was to move the L4/L7 load balancers. We needed the Load balancers to rewrite the destination headers and not need to be in the middle of everything. Part of the reason we did this is because I was offended by L2 LBs, and thought it was extremely hard to debug what was going on. Also we had several DDoS attacks and the LBs we had were incapable of defending us from external attacks and so we brought in a new LB that did L4/L7. So getting LBs out of the direct and no longer being the default gateway gave us a lot more flexibility. Flexibility that we wouldn't have survived without, but at the time, we didn't know how badly we'd need it. We just knew that they way we were working wasn't working.

That effectively got rid of most of the L2 requirement. Except that also each datacenter host had those two NICs that needed to be in two of three networks. Which either would require custom cabling or something dynamic like vlans. Either way required host configuration in the network that was dependent on the software that would be running on the host. If you can help it, you don't want networking in the way of the software that is running on the host. How did we solve this? We want back to the software teams and asked if we could just have each host have one NIC on one network. We got rid of the dbnet and had just a website and corporate network. It was fairly easy when the hosts were ordered to know website or corporate, so this was not a big burden. This is an example of understanding the tradeoffs. The business originally asked for two NICs probably in support of better security, but that wasn't really true and it lead to more operational pain which lead to more outages. It's certainly easier to have this argument after you have years of struggling to keep the network running. At this point it's easier to argue that the network needs more complexity.

## Multicast

As I've [written about before](https://elegantnetwork.github.io/posts/Lessons-from-load-balancers-and-multicast/), Amazon was a huge multicast user in 2002-2006. This was because the of the explosion in what we called service oriented architecture, but is now called microservices. The main framework used for these microservices used a publish-subscribe methodology to communitacate between most services. This was really cool, almost magic, but in the end it was a scaling disaster. It took a while for us to realize that abstracting traffic between services this way would not scale and was problematic. It helped that there were software problems with all this. When it became clear things couldn't work this way we very quickly switched to using LBs between services. This was again a time that we had to go back to software teams. However, in this case we had mutlple outages and we were facing complete anialization as we rocket towards hard scaling cliffs in the hardware we had in the network.

##




vlans/l2
multiple nics
multicast
L2 LBs
not having policy in the LBs, instead adding a proxy layer