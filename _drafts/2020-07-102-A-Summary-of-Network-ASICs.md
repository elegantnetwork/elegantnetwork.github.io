---
layout: post
comments: true
author: Justin Pietsch
title: A summary of Neworking ASICS
excerpt: There is a lot going on in the field of the highest throughput network ASICs.
description: Current High Throuhgput Merchant Silicon ASICS
---
There is a lot going on in the field of the highest throughput network ASICs. The ASICs that I'm focusing on our focused on very high throughput, and not as many features or buffers. These are driven by the hyperscalers and datacenters in general. https://perspectives.mvdirona.com/2011/07/consolidation-in-networking-intel-buys-fulcrum-microsystems/  

An Application Specific Integrated Circuit (ASIC) is purpose built for applications. In this case, these are purpose built to provide as much network throughput as possible. It was really started by Broadcom and Fulcrum in the mid 2000s. [Fulcrum Terabit Clockless Crossbar switch](https://www.hotchips.org/wp-content/uploads/hc_archives/hc15/3_Tue/2.fulcrum.pdf) is a document from 2003, just the beginning for Fulcrum. They were using 130nm technology and building 1M transistor parts. We'll talk about what those mean later in this post. This wasn't even ethernet, it's purpose was to be in the middle of other network (ethernet) processing chips and just be the switch. Later generations were L2, and then they created an L3 switch. Fulcrum ASICs (Alta) at 24 ports of 10G were how Arista got started.

These chips (and others with more functionality and/or lower speeds) are sometimes called merchant silicon, meaning that these vendors sell these chips to other parties which might build systems including software around them. All network vendors use these merchant silicon in at least some of their routers. These are also used by all the big cloud companies and anybody talking about disaggregated networks. 

## How they work and how they are different from chips in Chassis

For the most part, the ASICs that we are talking about here work as a single ASIC in the device. Meaning that there are other chips such as ethernet MACs and CPUs, but all the network forwarding decisions and buffering are done inside one chip.




 They are mostly shared memory architecture chips.


### Node size and transistor count

The ASICs we are focused on are almost as big as it's possible to make a chip with

Reticle limit is 32x26mm
https://www.quora.com/Why-does-a-CPU-GPU-chip-have-a-physical-size-limit#:~:text=There's%20one%20hard%20limit%20that,as%20the%20limit%20for%2012nm.


Node size, 

## Current ASICs available or soon Available

| Vendor | Family | Name | ports | speed | Aggregate Tbps | year|
|--------|--------|------|-------|-------|----------------|-----|
| Broadcom | Strata XGS | Tomahawk 3|  | 400 | | |
| Broadcom | Strata XGS | Tomahawk 4|  | 800 | | |
| Broadcom | Strata XGS | Trident 4 | 
| Broadcom | dune/??? |
| Cisco | Silicon One |
| Innoviam | Teralink |||||
| INnoviam | ||||
| Nephos | |||
| Barefoot | |||



Let's dive into the companies that are working on these ASICs and what's going on.

### Broadcom




.


Some of the trade offs being made around node size. 



Tomahawk vs trident



Do I need a programmable pipeline in these ASICs?

Why I think P4 is a waste of time? 

## Why does this matter?
Dinesh wrote about https://medium.com/the-elegant-network/the-effect-of-switch-port-count-in-clos-topology-51489e0aa4bc Broadcom Tomahawk and what that size means for how designs are created.


## Other resources around these chips
https://www.nextplatform.com/2020/05/11/innovium-stays-on-broadcoms-heels-with-teralynx-8-switch-chips/
https://www.nextplatform.com/2019/12/12/broadcom-launches-another-tomahawk-into-the-datacenter/
https://www.nextplatform.com/2018/04/30/feeding-the-insatiable-bandwidth-beast/
https://www.nextplatform.com/2019/09/30/innovium-boosts-switch-chip-performance-without-a-process-shrink/
https://www.barrons.com/press-release/PR-CO-20190611-906402?tesla=y
https://packetpushers.net/virtual-toolbox/list-merchant-silicon-manufacturers-chips/
https://www.barrons.com/press-release/PR-CO-20190611-906402?tesla=y
https://www.broadcom.com/products/ethernet-connectivity/switching/strataxgs
https://bm-switch.com/index.php/blog/nba740-705-nephos/ -- 2018
https://people.ucsc.edu/~warner/buffer.html
https://www.cisco.com/c/en/us/solutions/service-provider/innovation/silicon-one.html

https://www.nextplatform.com/2019/09/13/everyone-will-want-higher-bandwidth-and-more-ports-eventually/
https://www.sdxcentral.com/articles/analysis/ocp-summit-moving-on-from-snow-white-and-the-seven-dwarfs/2019/04/