---
layout: post
comments: true
author: Justin Pietsch
title: A Follow-up to High Speed Ethernet ASICs
excerpt: I've gotten some feedback from my post on high speed ASICs, so I thought I'd write a follow-up.

description: I got some feedback about these ASICs I'd like to address.
---

I've gotten some feedback from my [post on high speed ASICs](https://elegantnetwork.github.io/posts/A-Summary-of-Network-ASICs/), so I thought I'd write a follow-up. I've also updated the table that I had created in the previous post. If you haven't read the original, it's better than this, so read it first.

One of the things I was trying to get across is the various tradeoffs that these designs are making. For instance, it's interesting to see when companies move to smaller node size, or when companies use chiplets.

## Updated table
I fixed a typo and added Marvell.

<script src="https://gist.github.com/jopietsch/c1573518516af6071ae9cd0462ff0fd3.js"></script>

## Chiplet ##
I don't really understand the tradeoffs in chiplets, but I got some feedback about what I wrote in the previous post. Some of the reasons to not use chiplets is that they require more power and they cost more. I assume that is because of the connectors between the chiplets. On the reasons to use chiplets, I think it's because since they are smaller blocks of silicon, you get better yields. I also assume there is some less complexity in design because they are smaller, I don't know.

I did read that AMD is using chiplets to get over the reticular limit with their EPYC 32 core processors. crazazy.

* <https://www.mccoycomponents.com/blog/view/understanding-chiplet-in-one-article>
* <https://www.extremetech.com/computing/290450-chiplets-are-both-solution-and-symptom-to-a-larger-problem>
* <https://www.semiconductor-digest.com/2020/08/25/alchip-technologies-reveals-secrets-behind-reticle-size-design/>
* <https://www.tomshardware.com/news/amd-epyc-rome-7000-series-data-center-processor-zen-2-7nm,40108.html>
* <https://wccftech.com/tsmc-broadcom-cowos-platform-worlds-first-largest-reticle-size-interposer/>

## P4 ##
I wrote disparaging things about P4 for two reasons: The first is that I haven't found any good reason for P4 in switch ASICs, and so I'd like to be educated about a real reason for it. If I'm wrong, let me know. I'd like to see something in production or worthy of production. People can promise lots of cool things that P4 could do, but then the realities of the tradeoffs are not worth it. For instance, I think with P4 you could write a L4 load balancer with these ASICs. However, load balancing is much better at L7, so that doesn't do you much good. Also, I'm still unclear how state tracking works with these ASICs even for L4.

The other reason, and it's my primary reason, is that  I want real revolutions in networking, and instead we end up wasting so much time on things like P4 and Openflow that don't do what we need. I'm not bitter. At all.

I have heard that P4 (or Broadcom's NPL) is being used for extra telemetry. That's interesting. I don't think it lives up to the promise, but it is interesting.

Please let me know if you have actually seen P4 in switches/routers used in an actually useful way. 

## Chassis ##
Some of these single ASIC chips, such as Tomahawk, can be used in chassis. Broadcom XGS chips have a feature called HiGig which speeds up the interfaces and adds some proprietary headers so that when these chips can communicate they know where in the chassis they are. For the most part, if you are going to build a chassis you might as well use the more sophisticated chipsets as the DNX (Dune), but it is available. Remember, I'm not a hardware engineer, I don't really know the tradeoffs, but I do know that DNX like chips do have more features that can be important in a chassis.

I think chassis are bad ideas and you should move away from them as fast as you can, but that's for another blog post.

## Marvell ##

I skipped Marvel last time, and that was an error on my part. Marvell started in 1995, and has been in the network business for a long time. In my initial investigation, I did search for Marvell, but I thought that they didn't have chips in this category.I was wrong. Their current line is the Prestera line and they say it's their 9th generation switch pipeline.

There's not a lot of information or news on Prestera; it's hard to find any information that isn't just press release.

https://www.marvell.com/content/dam/marvell/en/public-collateral/switching/marvell-switching-prestera-98cx8500-product-brief-2019-03.pdf
https://www.servethehome.com/marvell-2020-networking-portfolio-update/ 


## Cisco Q100

Both a "routing" ASIC and a "switching", I'm not sure what that means

https://www.cisco.com/c/en/us/solutions/silicon-one/white-paper-sp-product-family.html#~one-architecture

## Xsight
https://www.businesswire.com/news/home/20201209005255/en/


## Conclusion
Let me know if I'm missing anything else, or wrong :)
