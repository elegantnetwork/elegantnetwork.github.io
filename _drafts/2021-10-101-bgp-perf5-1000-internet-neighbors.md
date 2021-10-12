---
layout: post
comments: true
author: Justin Pietsch
title: BGPerf 5: 1000 full internet neighbors
excerpt:
description: To test to the limit of how many neighbors these BGP stacks can have with full internet routes
---


- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Followup Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)
- 4th Post [Bird on Bird, Episode 4 of BGP Perf testing ](https://elegantnetwork.github.io/posts/bird-on-bird-bgp-perf-episode4)

In the 3rd Post we compared full internet stacks with up to 50 neighbors. What happens if we try to get 1000 Neighbors? Is that realistic? I don't know. I have heard of people with 1M L3VPN routes that want hundreds of neighbors.

I didn't think it would be useful, but then I started running experiments and found that I could break the stacks in interesting ways so I figured I should write up a blog post.

# Test setup
A major question is if it's reasonable to be able to test 1000 neighbors with full internet tables (800K prefixes). I wasn't sure, but [bgperf](https://github.com/jopietsch/bgperf) has support for playing back mrt files using [bgpdump2] which is very efficient.

My initial test device is my AMD system with 32 cores and 64 GB of RAM. It's not hard to run out of RAM on that platform, so I also used EC2 M5.24xlarge whichhad 96 cores and 384 GB of RAM and I didn't run out of RAM.


# FRR




