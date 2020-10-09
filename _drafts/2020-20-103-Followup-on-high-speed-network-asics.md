---
layout: post
comments: true
author: Justin Pietsch
title: A Followup to High Speed Ethernet ASICs
excerpt: 
description: I got some feedback I'd like to address on these ASICs.
---

I've gotten some feedback from my post on high speed ASICs, so I thought I'd write a followup.

One of the things I was trying to get across is the various tradeoffs that these designs are making. For instance, it's interesting to see when companies move to smaller node size, or when companies use chiplets.




## Chiplet ##
I don't really understand the tradeoffs in chiplets, but I got some feedback on them. Some of the reasons to not use chiplets is that they require more power and they cost more. I assume that is because of the connectors between the chiplets. On the reasons to use chiplets, I think it's because since they are smaller blocks of silicon, you get better yields. I also assume there is some less complexity in design because they are smaller, I don't know.

I did read that NVIDIA is using chiplets to get over the reticular limit. crazazy.


## P4 ##
I wrote disparaging things about P4 for two reasons: THe first is that I haven't found any good reason for P4 in switch ASICs, and so I'd like to be educated about a real good reason for it. I'd like to see something in production. People can promise lots of cool things that P4 could do, but then the realities of the tradeoffs are not worth it. For instance, I think with P4 you could write a L4 load balancer with these ASICs. However, load balancing is much better at L7, so that doesn't do you much good. Also, I'm still unclear how state tracking works with these ASICs.

THe other reason, and it's my primary reason, is that because I want real revolutions in networking, and instead we end up wasting so much time on things like P4 and Openflow that don't do what we need. I'm not bitter. At all.

## chassis ##
These single ASIC chips, such as Tomahawk, can be used in chassis. They have a feature called HiGig which speeds up the interfaces and adds some proprietary headers so that they chips can communicate to each other. For the most part, if you are going to build a chassis you might as well use the more sophisticated chipsets as the DNX (Dune), but it is available.


## Marvell ##

I skipped Marvel last time, and that was an error on my part. Marvel started in 1995, and has been in the network business for a long time. I did search for Marvell, but I thought that they didn't have chips in this category, but I was wrong. Their current line is the Prestera line and they say it's their 9th generation switch pipeline.



https://www.marvell.com/content/dam/marvell/en/public-collateral/switching/marvell-switching-prestera-98cx8500-product-brief-2019-03.pdf
https://www.servethehome.com/marvell-2020-networking-portfolio-update/ 