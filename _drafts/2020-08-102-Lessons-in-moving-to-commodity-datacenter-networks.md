---
layout: post
comments: true
author: Justin Pietsch
title: Lessons in moving to commodity datcenter networks
excerpt: This is my story on how Amazon moved to commodity networks, others will tell it differently.
description: Moving Amazon to disaggregated white box networking was a big challenge for us at Amazon in 2009-2012.
---
This is my story on how Amazon moved to commodity networks, others will tell it differently. We all think of amazon as gigantic, but in 2009 Infrastructure and Networking was small. 

James Hamilton is right. A Lot. **I don't like to admit that**, and I don't like to name drop, but he was very right. 

In June of 2009 I started a new job inside Amazon. I'd been focusing on infrastructure tools, not really in networking for a brief time and was even suckered into being a manager for network tools (4 employees). James had come to AWS in December of 2018 to cause trouble, and had started causing trouble in networking in the spring of 2009. At some point the powers that be told our VP to assemble his most senior network engineers and start thinking about greenfield datacenter networks. Two of us and our manager started working on this, in a very non-amazon way. We had to write papers, but we didn't have any specific outcomes and we couldn't really do a working backwards paper at this point.

**We were told we had to give a hard look at using commodity ASICs**.  James wrote up https://perspectives.mvdirona.com/2009/12/networking-the-last-bastion-of-mainframe-computing/ on a napkin. I don't know what would have happened if we had said we couldn't make it work. It took us quite a while to be convinced that we could make it work. Because we had had so much multicast (lb and multicast post) we always assumed we had to have big buffers. At some point I remember thinking, well, it's probably not going to work, and if it fails and I get fired at least it will have been fun and I'll have learned a lot.

Arista was around, preaching the good news of merchant silicon (merchant silicon blog)  and we talked to them, but in an Amazon way, we wanted to do it ourselves. **At the very least we wanted to be able to purchase the software separate from the hardware like in the server world.** This is now called disaggregated networking. At the time, there were two viable chips options, Fulcrum and Broadcom. We liked Fulcrum a lot, but broadcom had huge advantages for us. One of them was they had an OS, Fastpath, that worked.  You've probably never heard of Faspath, and that was rather terrifying. The second was that we could just order ODM white box equipment from the big Taiwanese ODM hardware companies. Fulcrum didn't have these abilities, and we didn't have any hardware design experience. We took whatever the ODMs had, which was really under-powered, but we didn't know that. **We made mistakes** like not having ECC memory, barely any storage, and truly under-powered CPUs.

At the time we had a fairly normal access - aggregation - core model, what James called Cisco Normal Form. It was all L3, we'd stamped out vlans and L2 in 2003. We also didn't have any virtualization in the network, EC2 originally did it's networking directly on the host, and by 2009 Amazon had an overlay network that was not in the physical network. We also already hated chassis routes, but thought we had to use them. Chassis are really complicated and fail in weird ways.

We hadn't thought about Clos, so we spent a lot of time drawing out Clos networks, comparing them to more sophisticated topologies and understanding the choices. There is a lot of interesting work in the supercomputer research, but in the end we understood that with all of the customer traffic from EC2 we had to assume random worse case traffic flow, which mean Clos was the winner. We had to then figure out how to come up with a plan on how to incrementally build out a big Clos network and also how to get routing protocols to work on it. (BTW whenever you hear of fancy topologies like dragonfly, or butterfly, or whatever, be very suspicious. It's unlikely they could be better than Clos.)

We were focusing on replacing the aggregation part of our datacenters. We wanted to create an aggregation fabric. We knew we'd still need a core layer to connect datacenters together and various policy layers to keep things clean. And just to be clear, what we were proposing was using hardware that until then had almost exclusively been used for access layer / ToR. We were going to replace large parts of the network with these devices. Rather than pairs of routers, we'd be replacing that with (eventually) a three tier Clos. At that point the hardware available was Broadcom Scorpion with 24 ports of 10G, and 16K of v4 LPM.

One of the key drivers, at least as far as James and other VPs were concerned was that we would make networking much cheaper. James spoke a lot about [getting networking out of the way.](https://mvdirona.com/jrh/TalksAndPapers/JamesHamilton_POA20101026_External.pdf) Networking usually costs somewhere between 10-15% of the total cost of a datacenter. Whenever you are making decisions to have cheaper networking and thus standing compute or storage resources you are making the wrong choice. But when networking costs so much it's hard to want to do that. I was much less interested in figuring out how to make capital costs lower. I was much more interested in figuring out how to build, scale, and operate these networks much better. Though if we could make it cheaper, it would be easier to argue for more hardware when more hardware could equal a simpler design. At the same time, when you go from chassis to pizza boxes, you need about 10x the number of devices, so you had better figure out a better way to operate the network. As stated above, we wanted to get the network out of the way for customers and for operators. The point wasn't really to use commodity devices, it was to build a network that would be our platform for growth, which meant it needed to be better to operate than the network we had.

We needed to have one network that could support all of EC2. If it was HPC, or storage or general compute, that didn't matter. It needed to be all the same network. 

By the fall we brought on a couple more people to make the first deployment real. We decided that we’d build out a first version that was specially built for a [new HPC offering in EC2.](https://perspectives.mvdirona.com/2010/11/aws-compute-cluster-231-on-top-500/) [In June of 2010](https://aws.amazon.com/about-aws/whats-new/2010/07/13/announcing-cluster-compute-instances-for-amazon-ec2/). This wasn't a Clos, it was, well, weird, but allowed us to try out these ideas, and the software and hardware stack that we were moving to. It was pretty small, only allowing a couple hundreds hosts. This network was turned off only in the last couple years🙂 

We had some big risks that we needed to mitigate, such as:
* Small table, tiny buffer hardware
* Fairly untested NOS in Fastpath. We'd never used it and didn't know of anybody big using it
* Very large scale OSPF in a dense mesh, which I already talked about
* 10x more hardware, cables, and optics than we'd had previously

Risks that we'd inherited from our very fast growing network.
* Hey, we are out of capacity, what should we do next?
* Almost no automation for configuration except for the ToR switches. No way to define designs and make sure that all deployments are the same.
* Failures and outages from config management and config changes.
* Chassis routers are more likely to grey / unpredictably fail than a single ASIC device.
* One network for all of EC2, not specialized for different workloads.


How did we mitigate these risks?
* make the rack of switches a SKU
* pretty strict config management that made everything regular
* simulation
* focused testing to destruction of the hardware
* very specific configuration. We knew what we needed and we removed any other functionality. We couldn't trust anything unless it had been tested. You couldn't just turn on a new feature.
* We spent way way more time designing and architecting this solution than we'd ever before.

How did we make it more operatable?
* Don't worry about using every single port. Maximum efficiency isn't the goal. Operatable is.
* Completely regular topology. If I could t describe it in an algorithm we couldn’t do it
* Though about how to scale it incrementally. We needed to grow the ports to the severs and also the bandwidth between them if necessary
* Minimize recabling. I think recabling is the worst, but we weren’t able to think ahead of everything. Plus our first version wasn’t big enough for some data centers, the hardware wasn’t what we needed at the time. 
* Assume devices, cables, and optics will fail and make sure those events don't immediately require a human to act.

We didn’t have nearly the tooling that we needed. Remember, Amazon networking was tiny. I’d built a config management system to produce the config but we didn’t have a good way to update it. We launched several deployments of parts of a 3 tier Clos before the software was ready to update, which was upsetting to the operators. We were trying to balance not buying any more chassis routers that we knew we didn't want, with a new network that wasn't completely ready. This is one of the biggest challenges when you are always growing is how do you take the time to architect something new and move to it.

A controversial decision and implementation that I think was hugely important but others don’t like: from the beginning we deployed these with full configs and then rebooted the devices for the changes. There are several reasons for this. First, I’d written the config tool to help me simulate the network, so it was naturally producing all config. Second, we’d had many outages over the years on which small config changes were not correctly applied and the device went crazy. When you think about All the state that a router has and then changing that state in arbitrary times in arbitrary ways is hard. It’s very hard to test, and the vendors had demonstrated they were testing well enough. What is tested well? Boot up with an arbitrary config. I believed we would skip a lot of bugs by doing this. Of course this also meant that the config system would be likely to be authoritative. 

Let me take a tangent and talk about design trade offs. We were going down some radically news paths to get something radically better. But whenever you do something so much different you are making trade offs. Some things will be worse, your goal 

Turn it into a product with a SKU. This was not my idea and I had nothing to do with it and it was really fantastic. We (not me) made it packagable so that we could get systems integrators to build racks offsite and ship them and then we had cable harness to make the insane cabling sane. TPMs can order these when capacity is needed and it doesn't require any new design work.

I don't think anybody else has been as aggressive As amazon in getting rid of chassis routers, or even big table routers. If you are going to do that and want to succeed, then you need to think about what that entails. As I mentioned, thinking of the network as a product was a big part of that. Figuring out how you will scale and grow is critical.

Have I mentioned I hate chassis routers?

What do I want people to learn:
* what it's like to do something big
* that there are a lot of tradeoffs, and to think through the tradeoffs
* it's possible to do commodity 
* the lessons I've learned and the story about them