---
layout: post
comments: true
author: Justin Pietsch
title: Comparing Open Source BGP stacks with internet routes
excerpt: 
description: 3rd episode testing BGP stacks, this time with internet routes.
---

- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Followup Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)

I hope it's clear by now that these posts are something of my lab notes as I explore how to test BGP stacks. I'm learning how to test as well as I'm learning how these perform. There are not yet comprehensive in any sense of the word; I'm still working on making the testing better. 

On the other hand, I couldn't find any useful performance results on the internet that already existed, so I wanted to at least get something out, as well as show my methodology (and publish my code) in the hopes if I got something wrong, I might be corrected or improved.

In the previous two posts, it was much about exploring limits across dimensions of prefixes and neighbors. Mostly trying to explore limits both of the testing and of the stacks. Because there are so many different uses of BGP stacks, there isn't one set of scenarios that are best to test. On the other hand, some aspects are not real world at all. The two most obvious are that it uses unique prefixes so that there is not best path comparison; this is especially glaring at internet size (1M+) tests. The other is that there is not any filtering. In this post I'm addressing the first, I hope to get to the filtering soon, just not yet.

Also, I do intend to add commercial stacks at some point, hopefully soon. I'm just struggling with guessing what would be most instructive to make better. I want to know how they compare also, I guess for now I'm more concerned about making the testing a little better before I do the effort of adding  more stacks. What you don't see in these results is that I end up spending a lot of time shepherding these tests. Sometimes things fail in unclear ways and bgperf isn't robust.

# Internet Route Tables Tests
These tests use real routing tables that have been captured from multiple routers. So in these tests, unlike the previous blog posts, the testers are not sending unique prefixes, and the targets have to do best path calculations. The most common format is called [MRT](https://datatracker.ietf.org/doc/html/rfc8050). [bgperf](https://github.com/jopietsch/bgperf) now has two different mechanisms to play back MRT files: bgpdump2 and GoBGP. In this post I used both because they have different characteristics.

I don't know how realistic it is to have 40-50 BGP peers with full BGP peers.
## bgpdump2 results
First we'll look at bgpdump2. bgpdump2 is really fast, so it can overwhelm the target.
![bgpdump elapsed time](/assets/images/2021-08-bgp-stacks-internet/bgperf_bgpdump-all_elapsed.png)

Lots of things that might jump out fast. RustyBGP and GoBGP are the slowest, also RustyBGP, GoBGP,  and Openbgpd don't have results for 30, 40, or 50 neighbors. BIRD is the fastest. FRR 8 is a bit faster than FRR 7.5.1. Why don't OpenBGP and RustyBGP have results over 20? Starting at 30 neighbors, for both OpenBGP and RustyBGP I could not get them to ever finish. I'll talk about that below. GoBGP was just so slow I stopped.

## GoBGP results

![GoBGP elapsed time](/assets/images/2021-08-bgp-stacks-internet/bgperf_GoBGP-MRT-all_elapsed.png)

Using GoBGP as the MRT generator is clearly slower then bgpdump2. Maybe that makes it more realistic, I don't know. What's interesting is that for 5 and 10 neighbors, FRRouting 7.5.1, 8.0 and BIRD are exactly the same, which means that that they can't go any faster. However, OpenBGPD and RustyBGP are slower. It's also interesting that I could test > 20 neighbors for OpenBGPD, everything completed. BIRD is again the fastest with FRR 7.5.1 and 8.0 just a little bit behind. OpenBGP is considerably slower with > 10 neighbors, 2-3 times slower. GoBGP is faster than RustyBGP, which is not expected.

## bgpdump2 resource utilization


![bgpdump max mem](/assets/images/2021-08-bgp-stacks-internet/bgperf_bgpdump-all_max_mem.png)

BIRD is the most efficient, but a factor of 2 over FRR, and the fastest, which is amazing. FRR 8 uses more memory than 7.5.1 (we saw that even more dramatically with 1000s of neighbors in the last set of tests) and is faster. Is that related? For the tests it completes OpenBGPD is the most memory hungry. RustyBGP is more efficient than the FRRs, but much much slower.

![bgpdump max cpu](/assets/images/2021-08-bgp-stacks-internet/bgperf_bgpdump-all_max_cpu.png)

These CPU graphs aren't really that interesting.


## GoBGP resource utilization

![GoBGP max mem](/assets/images/2021-08-bgp-stacks-internet/bgperf_GoBGP-MRT-all_max_mem.png)

Pretty similar results to those from bgpdump2

![GoBGP max cpu](/assets/images/2021-08-bgp-stacks-internet/bgperf_GoBGP-MRT-all_max_cpu.png)

Still not that interesting.

## The setup

The setup to these tests matter a lot because they are really stressing these stacks. You might remember, that I'm using[bgperf] to do the tests. bgperf has the idea of a target (the stack being tested), a monitor that connects to the target and is advertised all the routes and counts how many have been received, and a set of testers. 

The bgpdump2 tests are again done on my AMD system with 32 cores and 64GB RAM. The tests with GoBGP as the generator were done on an EC2 m5.24.xlarge with 96 cores, and 384GB of RAM


The monitor is GoBGP. The role of the monitor is really interesting in these tests, specifically with bgpdump2. bgperf now checks to see on the target that each neighbor has sent all the expected # of prefixes and the test doesn't pass until both the monitor has received the expected number and has the target from each neighbor. In some tests, definitely with RustyBGP, I noticed that all the neighbors were done, but then much later would the monitor receive all the routes.


The way these works is that each of the testers is in a separate containers. All of the containers are started, without starting the MRT generator, and the each generator is started and there is a sleep between generators, just to be a little less severe to the target.

## MRT

As mentioned MRT is the standard way of recording internet routing tables. [Routeviews](http://www.routeviews.org/routeviews/) is a collection of people who voluntarily capture MRT data of the internet routes. You can grab their data. Inside one of these MRT files are the route tables of several different routers from many different ASes at the same time.

## bgpdump2

bgpdump2 is pretty much just an MRT playback cannon. Very fast.

I did find some interesting things when getting bgpdump2 to play back. When I first set it up, I had it sending from an arbitrary AS and no next-hop-self rewriting. This worked fine for FRR and RustyBGPd, but not BIRD or OpenBGPD. BIRD required both to work, OpenBGPD needed the correct AS to work, but not the next-hop-self. I don't know what any of that means, but I did find it.

## GoBGP
GoBGP uses a lot more resources which is why I ran it on the EC2 instance, so that the tests wouldn't run out of resources.

# RustyBGP

With bgpdump2, I got inconsistent results with 20 or more neighbors. Some neighbors never established connection. 

RustyBGP also blocks a lot when trying to get the neighbor data, even if there is available CPU on the machine.  bgperf tries to get neighbor info every second, but also I sometimes run commands to get the data to see what is going on, and it will block for 10s of seconds.



I don't know what to do about RustyBGP. It's hard for me to test and I'm not getting similar results (at all) to the creator. [https://twitter.com/brewaddict/status/1425607915080097800](https://twitter.com/brewaddict/status/1425607915080097800)


Theres' a lot that are different in our testing.

https://github.com/fujita/misc/tree/master/fullroute-bench/update-watcher

# OpenBGPD
With bgpdump2, Openbgpd would connect all 30 neighbors, run for a long time, get to where the target had received full prefixes (800k) from 29 of the 30 neighbors, and then the prefixes would drop to 0 for 2-3 neighbors and never increment. I ran this at least 3 times with the same results. I don't think the neighbors timed out. I do not know what was going on. Because bgpdump2 plays back so fast, I wonder how much this is artificial, but BIRD and FRR do fine with this. In the tests with GoBGP OpenBGPD doesn't have this problem.



# Conclusions
Is bgpdump2 or GoBGP MRT playback more realistic? Don't know, but the different results are interesting.

I mentioned that these are more realistic than my previous tests. That is definitely true for internet routes. However, if you have a RouteServer with > 1000 neighbors, they might all have unique prefixes, and so those tests might be realistic for that scenario.


There is a big mismatch between what I am seeing from RustyBGP and what Fujito is seeing.

## Questions
 I don't the impact of the monitor on these results