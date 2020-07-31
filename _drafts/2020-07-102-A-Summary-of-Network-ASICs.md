---
layout: post
comments: true
author: Justin Pietsch
title: A summary of High Speed Ethernet ASICs
excerpt: There is a lot going on in the field of the highest speed network ASICs.
description: Current High Speed Merchant Silicon Ethernet ASICs
---
There is a lot going on in the field of the highest speed network ASICs. The ASICs that I'm focusing on our focused on very high speed, and not as many features or buffers. These are driven by the hyperscalers and the Financial industry which desires the lowest latency switching. This is a very rich area and I've already written a lot. There's a lot more to dive into, but hopefully this will give you a taste and somethings to chase after if you want more understanding.

An Application Specific Integrated Circuit (ASIC) is purpose built for a particular purpose. In this case, these are purpose built to provide as much network throughput as possible. These ASICs can handle 10s of Billions of packets per second, where an Intel Xeon is on the order of hundreds of millions. They are two orders of magnitude faster than Intel when forwarding packets.

The use of these high speed ASICs in networking was really started by Broadcom and Fulcrum in the mid 2000s. [Fulcrum Terabit Clockless Crossbar switch](https://www.hotchips.org/wp-content/uploads/hc_archives/hc15/3_Tue/2.fulcrum.pdf) is a document from 2003, just the beginning for Fulcrum. They were using 130nm technology and building 1M transistor parts. We'll talk about what those mean later in this post. This chip wasn't even ethernet, it's purpose was to be in the middle of other network (ethernet) processing chips and just be the switch. Later generations were L2, and then they created an L3 switch. Fulcrum ASICs (Alta) at 24 ports of 10G were how Arista got started.

These chips (and others with more functionality and/or lower speeds) are sometimes called merchant silicon, meaning that these vendors sell these chips to other parties which might build systems including software around them. All network vendors use these merchant silicon in at least some of their routers. These are also used by all the big cloud companies and anybody talking about disaggregated networks. 

[Over half of the Ethernet switches that ship now, ship with Merchant Silicon](https://www.datacenterknowledge.com/cisco/why-cisco-got-merchant-silicon-market).
![switches shigped per year](https://www.datacenterknowledge.com/sites/datacenterknowledge.com/files/omdia%20ihs%20switch%20silicon%20q3%202019.png)

## How they work and how they are different from chips in Chassis

For the most part, the ASICs that we are talking about here work as a single ASIC in the device. Meaning that there are other chips such as ethernet MACs and CPUs, but all the network forwarding decisions and buffering are done inside one chip. Many people think of these as the chips for ToR switches, not the rest of the network, but some networks, and especially hyperscalers are using them throughout the network, usually in the form of small fixed form factor (pizza boxes.) Some people do combine them in interesting ways such as [Facebook's minipack](https://engineering.fb.com/data-center-engineering/f16-minipack/).

In any chip design there is a fixed budget of resources, which are transistors or space on the chip die and the number of pins from the chip to the rest of the system. If you want more of one thing, than you have to give up something else. In these chips, they are optimized to the most speed, so they have a lot of their die tied up in getting off the chip to the interfaces. One of the consequences of this is that they have much less buffering and table space than other chips. You might say well "Why can't they use off chip TCAM or buffers?", and there are two reasons. The first is that to use off chip resources means that you are using pins and chip real-estate that you wanted to use for providing more interface speed. 


There are chipsets that work together to form large and there are even merchant silicon than does this. The most used is the Broadcom Strata DNX. For instance, the [Arista 7500](https://www.arista.com/en/products/7500r-series) is [based on this chipset](https://www.arista.com/assets/data/pdf/7500E-Lippis-Report.pdf.) There are two big advantages to kind of chipset: they work together to build a larger system to support more ports and because they are not a single chip, they can handle offchip table and buffer space, so these have 1M+ table space and large buffers. 

## Important architecture considerations

There are many different designs that are chosen and we want to understand the impact of those.

### Node size and transistor count

Effectively all of the companies that build these chips are called Fabless, which means that they design the chips, but they don't physically create them. A Fab is the Fabrication plant that these chips are made. The largest is [TSMC](https://www.tsmc.com/english/default.htm), which I think makes most of these chips, if not all.

The ASICs we are focused on are almost as big as it's possible to make a chip with. It turns out that there is a limit to the size of chip that can be made in state-of-the-art Fabs which is called the [Reticle limit](https://www.quora.com/Why-does-a-CPU-GPU-chip-have-a-physical-size-limit#:~:text=There's%20one%20hard%20limit%20that,as%20the%20limit%20for%2012nm) which is 33x26 mm. I do not understand the physics or any of this process to understand why that it's but it means that the absolute max size is 858mm^2. [Nvidia's next gen chip GA100](https://developer.nvidia.com/blog/nvidia-ampere-architecture-in-depth/#:~:text=Fabricated%20on%20the%20TSMC%207nm,die%20size%20of%20826%20mm2.) is 54.2 billion transisotors and a die size of 826mm^2.

Another important number to understand the the [process node size](https://www.pcmag.com/encyclopedia/term/process-technology) a measure of the size of the transistors. Current state-of-the-art is 7nm. The advantage of smaller node size the more transistors you can get on the chip. In the past several decades Intel has always had the smallest process node size, until the last year. Smaller node sizes always go to the biggest money maker and so for TSMC and other foundaries like it those are now mobile processors. So 7nm mobile phone CPUs came out first. These are not big chips, but there are Billions of them. Soon after comes GPUs which are about as big of chips as you can make. And soon after that are these chips which are as big as the GPU.

One aspect of all this comes out in the amount of power that a chip consumes. Some of these are very power hungry, which makes them hard to power and harder to cool.

The design of these chips take a lot of money. It's very interesting how many different companies are trying to compete at this time and how much money some of them have raised. 

It's also interesting to note when a company moves to a smaller process size and when another company stays at the size they were using before. Moving to new node sizes adds a lot of risk, so companies mitigate that risk compared to other changes their are making in their designs.

### IP and SERDES

When talking about Integrated Circuits (ICs), IP means something different than most of us think of. It means Intellectual Property, but what that means is [it's a resuable unit of logic](https://en.wikipedia.org/wiki/Semiconductor_intellectual_property_core). For instance, ARM sells IP blocks to all it's customers to make all of the CPUs that everybody uses in their mobile phones.

But you don't have to buy the IP block for the whole chip. For instance, there's a really important IP block of these chips called the [Serializer/Deserializer (SerDes)](https://en.wikipedia.org/wiki/SerDes). This is the part of the chip that gets data in/out of the chip to the pins. Not everyone of these companies makes their own SerDes, many actually get their SerDes from Broadcom or Avago, which is now also Broadcom. It's interesting to try to keep track of how much influence Broadcom has.

### Chiplets

One way to keep up with the complexity and size of these chips is to break them up into what is called a [chiplet](https://en.wikichip.org/wiki/chiplet). These are [blocks of IP that are then put together in an end product](https://www.youtube.com/watch?v=f-4hxNKvEY0). Since they are smaller than one chip, you can take advantage of the fact that you will suffer less effects from manufacturing errors. Apple and other ARM customers have been doing chiplets for several years, and [AMD is now doing that in their CPUs](https://www.wired.com/story/keep-pace-moores-law-chipmakers-turn-chiplets/).

Some of these designs are using chiplets and some are not. ![Chiplets](https://eenews.cdnartwhere.eu/sites/default/files/styles/inner_article/public/sites/default/files/images/exascaleheterogeneous630.jpg?itok=VwcU0b9X) 

[These are the way of the future for many IC designs](https://www.eenewsanalog.com/news/birth-chiplet-market-shows-more-40-annual-growth).




### Pipelines and Buffers
Just like in CPUs, we've gotten to a place where you have to have multiple cores to take advantage of the transistors available, these ASICs have multiple pipelines that are interacting together to get the performance required. This is interesting to track because some of these chips seem to have more scalable pipelines than others based on the company's ability to continue to make a faster chip as time goes on.

Another important piece of pipelines in the last half-decade is how programmable the pipeline is. Barefoot was the first to come out with a programable pipeline based on [P4](http://www.sigcomm.org/sites/default/files/ccr/papers/2014/July/0000000-0000004.pdf). Broadcom has it's own language called [NPL](https://nplang.org/npl/blog/). 

As far as I'm concerned this does not matter. It sounds fantastic to have a programmable pipeline, but it's not actually necessary or even useful. I worked in Amazon/AWS Networking for 17 years and we neither wanted nor could make use of this functionality. I have not seen anybody actually use this functionality to do something that I thought was useful. But it sounds cool. [In this article about Xplaint](https://www.sdxcentral.com/articles/news/marvell-nixes-the-programmable-xpliant-chip-it-inherited-from-cavium/2018/08/) which also had a programmable pipeline, they talk about none of the hyperscalers find value in programmable pipelines. These chips are no longer made. 

* <https://bm-switch.com/index.php/blog/whitebox_basics_programmable_fixed_asics/>

* <https://www.nextplatform.com/2018/12/04/programmable-networks-get-a-bigger-foot-in-the-datacenter-door/>

## It's not just the hardware

These are very complex chips. I do not know how hard it is to integrate the hardware of the different chips. But I do know that the different SDKs integration are a major challenge. Many of the vendors try to mimic the Broadcom SDK because then software systems have an easier chance to integrate, since Broadcom has been around the longest and has the most extensive portfolio.

Supporting multiple SDKs is actually very hard work, so as you are choosing your ASIC, you need to be thinking if they have a future or not. Broadcom seems to have [open sourced their SDK](https://www.broadcom.com/products/ethernet-connectivity/software/sdklt), probably in an attempt to make it even harder to switch to other vendors



## Current ASICs available or soon Available

The dates are when the company says they are available, usually to select customers. That's different from when they are available in products. There is some deep competition here, and depending on the year one company comes out with the fastest ASIC soonest. It's hard to find all this information, especially from publicly available sources, which is what I've done here. I've found as much information as I could.

<script src="https://gist.github.com/jopietsch/c1573518516af6071ae9cd0462ff0fd3.js"></script>




### Barefoot / Intel

Barefoot was a startup focused on high speed ethernet ASICs with a programmable pipeline. They and some researches crated P4. The intention, as far as I understand, is that with SDN/Openflow you want a programmable pipeline so that you can get the most out of the resources. 

I cannot find an announcement for anything past Tofino 2 in 2018

This is not Intel's first foray into networking. They've done it several times. The last time was [when they bought Fulcrum](https://perspectives.mvdirona.com/2011/07/consolidation-in-networking-intel-buys-fulcrum-microsystems/) and then nothing relevant came out of Fulcrum after that acquisition. They have also purchased Network Processor Unit (NPU) companies in the past. [Not much ever comes out of Intel after these acquisitions](https://www.nextplatform.com/2019/06/11/the-games-a-foot-intel-finally-gets-serious-about-ethernet-switching/), though others are not so pessimistic as I am.


I'm biased against Barefoot/Intel. Barefoot sees themselves as a premium brand and was trying to charge premium price because of their programmable pipeline, but I do not see the need for programmable pipeline. And then Intel's history with Fulcrum makes me not trust the future. That's just opinion, not data, so take it for what it's worth. I'm still salty over the fulcrum's disappearance after being purchased by Intel. For a while there was no real rival to Broadcom and that isn't good for the industry.

* <https://www.nextplatform.com/2018/12/04/programmable-networks-get-a-bigger-foot-in-the-datacenter-door/>
* <https://www.barefootnetworks.com/products/brief-tofino-2/>

### Broadcom

Broadcom is the giant here, the king, the 800 pound gorilla. Broadcom was bought by Avago several years ago, and the whole company was renamed Broadcom.

Broadcom has two important families of Chips in this space. One is the Strata XGS, the other is Strata DNX which was from a company Broadcom purchased called Dune. The XGS are the single ASIC type that we've been talking about in this article. The Dune chipset can be used for chassis devices or in fixed-form-factor if you need large tables. The Jericho 2 is the chip that can be used standalone or with a fabric chip, which is called the [FE9600](https://www.broadcom.com/products/ethernet-connectivity/switching/stratadnx/bcm88790)

Even in the XGS line there are several sub-species. The one the hyperscalers focus on is Tomahawk, which focused on speed. The Trident line can more features, I think they call this an edge services line. They have even more chips ethernet chips, but we aren't be concerned about them for this article

Broadcom is a **very** aggressive company. They are not always fun to be a partner with. However, they are the biggest so you have to always be keeping track of them. Since my friend [Ian Cox](https://techfieldday.com/appearance/broadcom-presents-at-networking-field-day-22/) they seem to have gotten themselves back in shape! :)

Broadcom is like Intel in the CPU space. They go in waves in which they are the clear leader and there is almost no competition, then they seem to slow down and competition breaks out. We are at that point in the cycle right now, though Broadcom has stepped up their game in the latest Tomahawks to really put pressure on their competition. This is very good for the industry.

![Broadcom lineup](https://3s81si1s5ygj3mzby34dq6qf-wpengine.netdna-ssl.com/wp-content/uploads/2019/12/broadcom-asic-families.jpg)

![Tomahawk lineup](https://3s81si1s5ygj3mzby34dq6qf-wpengine.netdna-ssl.com/wp-content/uploads/2019/12/broadcom-tomahawk-roadmap.jpg)

* <https://www.nextplatform.com/2019/12/12/broadcom-launches-another-tomahawk-into-the-datacenter/>
* <https://www.broadcom.com/products/ethernet-connectivity/switching/strataxgs>
* <https://www.broadcom.com/news/product-releases/broadcom-ships-jericho2>
* <https://www.broadcom.com/company/news/product-releases/52196>
* <https://www.nextplatform.com/2019/06/24/bringing-big-bandwidth-to-large-enterprises/>

### Cisco

In case you didn't know, Cisco is selling merchant Silicon chips. They [bought a company Called Leaba](https://www.cisco.com/c/en/us/about/corporate-strategy-office/acquisitions/leaba.html), which was started by several of the founders of Dune. In general I would be very wary of using Cisco as a provider, they've not done this before, but the Dune/Leaba team is really good which is why this is interesting to me. I'm not sure how this will play out. It will be especially interesting to see how Cisco figures out to support an SDK to external customers.

The Q100 is more like the Strata DNX/Dune family than the Tomahawk. It's also used in products at Cisco including the [8200](https://www.cisco.com/c/en/us/products/routers/8000-series-routers/index.html)

Cisco, of course has many different inhouse ASICs that they don't sell to third parties. And they have several switch lines that use other Merchant Silicon.

* <https://www.datacenterknowledge.com/cisco/why-cisco-got-merchant-silicon-market>
* <https://www.cisco.com/c/en/us/solutions/service-provider/innovation/silicon-one.html>

### Innovium
 
Innovium is a very aggressive startup in which the main people from Innovium came from Broadcom. It is a very good team of people. 
It's interesting that Innovium is [winning designs (https://www.convergedigest.com/2019/06/innovium-silicon-powers-two-cisco-nexus.html) even though Broadcom as really upped their game in the last couple of years.

Innovium [has raised $350M Dollars](https://www.zdnet.com/article/network-chip-contender-innovium-scores-170-million-to-challenge-broadcom-prepares-for-the-400-gig-onslaught/). I don't know enough about these to know why it has required so much money, nor why investors think they can gain that back, other than they are winning designs.

![Innovium lineup](https://3s81si1s5ygj3mzby34dq6qf-wpengine.netdna-ssl.com/wp-content/uploads/2020/05/innovium-teralynx-8-serdes-adoption.jpg)

* [TeraLynx 8](https://www.innovium.com/wp-content/uploads/Innovium_TL8_Product_Brief_v1.0.pdf)
* [TeraLynx architecture](https://www.innovium.com/wp-content/uploads/Highly-Scalable-TERALYNX-Architecture-Whitepaper_v2.pdf)
* <https://www.nextplatform.com/2020/05/11/innovium-stays-on-broadcoms-heels-with-teralynx-8-switch-chips/>
* <https://www.zdnet.com/article/network-chip-contender-innovium-scores-170-million-to-challenge-broadcom-prepares-for-the-400-gig-onslaught/>
* <https://www.nextplatform.com/2019/09/30/innovium-boosts-switch-chip-performance-without-a-process-shrink/>
* <https://www.nextplatform.com/2018/06/20/a-deep-dive-into-ciscos-use-of-merchant-switch-chips/>
* <https://people.ucsc.edu/~warner/Bufs/innovium.pdf>

### Mellanox
Mellanox is really know as the InfiniBand company, but they also sell Ethernet chips. Mellanox was purchased by Nvidia this year.

* <https://www.nextplatform.com/2020/03/26/mellanox-doubles-up-ethernet-bandwidth-with-spectrum-3/>
* <https://www.mellanox.com/files/doc-2020/pb-spectrum-3.pdf>



### Nephos

[Nephos](http://www.nephosinc.com/about-us/) is a spinout of Chinese chip giant MediaTek, focusing on these switch ASICs. They were very aggressive several years ago, but haven't kept up with the competition. They don't seem to have a future, but I don't know. We'll have to see. I heard a rumor that they were being spun back into MediaTek, but I can't confirm that at all.


## Why does any of this matter?
The number of ports you have available can effect your design. @Dinesh wrote about [Broadcom Tomahawk and how the port size effects design](https://medium.com/the-elegant-network/the-effect-of-switch-port-count-in-clos-topology-51489e0aa4bc). 

It's also interesting to see how the industry moves. The more competition for Broadcom (and Cisco) the better for all of us.


## Other resources around these chips

I've included links to resources throughout the post. I'd recommend the <http://www.nextplatform.com> specifically, they have a wealth of information and are generally fantastic.


* <https://www.nextplatform.com/2018/04/30/feeding-the-insatiable-bandwidth-beast/>
* <https://packetpushers.net/virtual-toolbox/list-merchant-silicon-manufacturers-chips/>
* <https://people.ucsc.edu/~warner/buffer.html>
*  <https://www.nextplatform.com/2019/09/13/everyone-will-want-higher-bandwidth-and-more-ports-eventually/>
* <https://www.sdxcentral.com/articles/analysis/ocp-summit-moving-on-from-snow-white-and-the-seven-dwarfs/2019/04/>


## Comments
As an aside, this should be some kind of knowledge base or wiki, but I don't know what that would be.

Let me know if I'm missing anything and/or I got something wrong. And updated sources would be fantastic.