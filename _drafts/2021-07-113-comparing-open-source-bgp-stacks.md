---
layout: post
comments: true
author: Justin Pietsch
title: Comparing Open Source BGP Stacks
excerpt:
description: Compare some simple performance characteristics of three Open Source BGP stacks: BIRD, FRRouting, and Gobgp.
---

BGP stacks are important. I think open source BGP stacks are very important. There's lots going on in open source BGP stacks and I can't keep up. So I thought I'd like to quantitatively compare them. This is one, often tiny, aspect of evaluating a BGP stack. I didfairly simple testing. Very little policy, just the number of routes and/or number of neighbors are the independent variables.

The stacks evaluated are BIRD, FRRouting, GoBGP. These have different sets of features, including FRR and BIRD have more full routing stacks.


I started with [BGPerf](https://github.com/osrg/bgperf), written by the same people that write gobgp. BGPerf hasn't been updated in 4+ years, doesn't work with Python3 and doesn't correctly configure any of the current versions of these protocol stacks. I've [forked and updated BGPerf](https://github.com/jopietsch/bgperf) to actually work, at least for the tests I ran. BGPerf has quite a bit of complexity that I didn't try out, especially around remote test subjects.

BGPerf uses exabgp to create and send all the routes. In some of the tests, exabgp adds a lot of load to the test because it's not the most efficient code (written in Python). However, I think you can see what this means by comparing the same test for the three different stacks.

## What do these results mean?
The way that BGPerf works is that it creates all the configuration needed for the tester, the stack under test, and the observer. If you have more than one "neighbor", that means it's more than one ExaBGP process. There are lots of things that are going on that we have to measure. We have to measure how long it takes for exaBGP to get started, the time the test stack takes to initiate all the neighbor connections, and then the time it takes to pass all the routes.

BGPerf creates all the config in memory before writing it out, so if you have a large number of routes, like 10M, it will run out of memory on the 32bit Python process before it even gets to any of the rest of the test. This clearly could be improved.

Let's look through some examples. My tests so far are primarily done on my 16 core AMD 3970 with 64 GB of RAM. I did some tests also on an EC2 m6g.16xlarge.



###



## Observations
I was assuming that gobgp would use more CPU resources but be quite a bit faster on fast hardware. It turns out it just uses a lot of CPU resources and is a lot slower.


## Conclusion / Followup
I'd sure love it if you want to have a discussion about how to have better tests. If you propose a test, I'd love config snippets for the protocol stacks that show exactly what you want to compare. Or even better, PRs to BGPerf


## All Results

<script src="https://gist.github.com/jopietsch/9ce29828c7faca9678a499dc942248f6.js"></script>