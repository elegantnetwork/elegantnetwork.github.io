---
layout: post
comments: true
author: Justin Pietsch
title: Comparing Open Source BGP Stacks
excerpt: 
description: 
---

BGP stacks are important. I think open source BGP stacks are very important. There's lots going on in open source BGP stacks and I can't keep up. So I thought I'd like to quantitatively compare them. THis is one, often tiny, aspect of evaluating a BGP stack.

The stacks I'm evaluating are BIRD, FRRouting, GoBGP. These have different sets of features, including FRR and BIRD have more full routing stacks. 

I'm doing fairly simple testing. Very little policy, just straight the number of routes and/or number of neighbors are the independent variables.

I started with [BGPerf](https://github.com/osrg/bgperf), written by the same people that write gobgp. BGPerf hasn't been updated in 4+ years, doesn't work with Python3 and doesn't correctly configure any of the current versions of these protocol stacks. I've [forked and updated BGPerf](https://github.com/jopietsch/bgperf) to actually work, at least for the tests I ran. BGPerf has quite a bit of complexity that I didn't try out, especially around remote test subjects. Bu

BGPerf uses exabgp to create and send all the routes. In some of the tests, exabgp adds a lot of load to the test because it's not the most efficient code (written in Python). However, I think you can see what this means by comparing the same test for the three different stacks.




## Conclusion / Followup
I'd sure love it if you want to have a discussion about how to have better tests. If you propose a test, I'd love config snippets for the protocol stacks that show exactly what you want to compare. Or even better, PRs to BGPerf


## All Results

<script src="https://gist.github.com/jopietsch/9ce29828c7faca9678a499dc942248f6.js"></script>