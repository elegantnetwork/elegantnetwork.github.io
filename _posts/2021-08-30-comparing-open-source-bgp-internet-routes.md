---
layout: post
comments: true
author: Justin Pietsch
title: Comparing Open Source BGP stacks with internet routes
excerpt: In this post I'm addressing the the issues of testing with real internet routes so we get to stress the best path algorithm
description: 3rd episode performance testing BGP stacks, this time with internet routes.
---

- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Followup Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)

I hope it's clear by now that these posts are something of my lab notes as I explore how to test BGP stacks. I'm learning how to test as well as I'm learning how these perform. There are not yet comprehensive in any sense of the word; I'm still working on making the testing better. On the other hand, I couldn't find any useful performance results on the internet that already existed, so I wanted to at least get something out, as well as show my methodology (and publish my code) in the hopes if I got something wrong, I might be corrected or improved.

The two previous posts were much about exploring limits across dimensions of prefixes and neighbors. Mostly trying to explore limits both of the testing and of the stacks. Because there are so many different uses of BGP stacks, there isn't one set of scenarios that are best to test. On the other hand, some aspects are not real world at all. The two most obvious are that it uses unique prefixes so that there is not best path comparison; this is especially glaring at internet size (1M+) tests. The other is that there is not any filtering. In this post I'm addressing the the issues of testing with real internet routes so we get to stress the best path algorithm, I hope to get to the filtering soon, just not yet.

I do intend to add commercial stacks at some point, hopefully soon. I'm just struggling with guessing what would be most instructive to make better. I want to know how they compare also, I guess for now I'm more concerned about making the testing a little better before I do the effort of adding more stacks. What you don't see in these results is that I end up spending a lot of time shepherding these tests. Sometimes things fail in unclear ways and bgperf isn't totally robust and needs to be watched.

# Internet Route Tables Tests

These tests use real routing tables that have been captured from multiple routers. Unlike the previous blog posts, the testers are not sending unique prefixes, and the targets have to do best path calculations. The most common format is called [MRT](https://datatracker.ietf.org/doc/html/rfc8050). [bgperf](https://github.com/jopietsch/bgperf) now has two different mechanisms to play back MRT files: bgpdump2 and GoBGP. In this post I used both because they have different characteristics.

I don't know how often people have 40-50 BGP peers with full BGP internet routes. I'm still trying to understand the performance edges of these stacks and how they compare against each other.
## bgpdump2 results

First we'll look at bgpdump2. bgpdump2 is really fast, so it can overwhelm the target.
![bgpdump elapsed time](/assets/images/2021-08-bgp-stacks-internet/bgperf_bgpdump-all_elapsed.png)

Lots of things that might jump out fast. RustyBGP and GoBGP are the slowest, also RustyBGP, GoBGP, and OpenBGPD don't have results for 30, 40, or 50 neighbors. Why don't OpenBGPD and RustyBGP have results over 20? Starting at 30 neighbors, for both OpenBGPD and RustyBGP I could not get them to ever finish. I'll talk about that below. GoBGP was just so slow I stopped. BIRD is the fastest. FRR 8 is a bit faster than FRR 7.5.1.

Again, this is testing something pretty extreme. I don't know how often you'd be happy waiting many minutes for full convergence.

## GoBGP results

![GoBGP elapsed time](/assets/images/2021-08-bgp-stacks-internet/bgperf_gobgp-mrt-all_elapsed.png)

Using GoBGP as the MRT generator is clearly slower then bgpdump2. Maybe that makes it more realistic, I don't know. What's interesting is that for 5 and 10 neighbors, FRRouting 7.5.1, 8.0 and BIRD are exactly the same, which means that that in those tests it's pretty much just testing the tester and not the stacks. However, OpenBGPD and RustyBGP are slower. It's also interesting that I could test > 20 neighbors for OpenBGPD, everything completed. BIRD is again the fastest with FRR 7.5.1 and 8.0 just a little bit behind. OpenBGPD is considerably slower with > 10 neighbors, 2-3 times slower. GoBGP is faster than RustyBGP, which is not expected.


## bgpdump2 resource utilization

![bgpdump max mem](/assets/images/2021-08-bgp-stacks-internet/bgperf_bgpdump-all_max_mem.png)

BIRD is the most efficient, but a factor of 2 over FRR, and the fastest, which is amazing. FRR 8 uses more memory than 7.5.1 (we saw that even more dramatically with 1000s of neighbors in the last set of tests) and is faster. Is that related? For the tests it completes OpenBGPD is the most memory hungry. RustyBGP is more efficient than the FRRs, but much much slower.

![bgpdump max cpu](/assets/images/2021-08-bgp-stacks-internet/bgperf_bgpdump-all_max_cpu.png)

These CPU graphs aren't really that interesting.


## GoBGP resource utilization

![GoBGP max mem](/assets/images/2021-08-bgp-stacks-internet/bgperf_gobgp-mrt-all_max_mem.png)

Pretty similar results to those from bgpdump2

![GoBGP max cpu](/assets/images/2021-08-bgp-stacks-internet/bgperf_gobgp-mrt-all_max_cpu.png)

Still not that interesting.

## More reasonable Tests

In case you aren't trying to peer with 50 peers with full internet routes, it's hard to understand the performance difference from the above tests. So let's do some more reasonable tests and see what we see.

This first test is with bgpdump2 as the tester.

![bgpdump reasonable tests](/assets/images/2021-08-bgp-stacks-internet/bgperf_bgpdump-reasonable-internet_elapsed.png)

It's interesting the FRR, BIRD, and OpenBGPD are about the same at 1 neighbor. This makes me think that the limitation is in the bgpdump2, but maybe not. It's then interesting the BIRD takes barely any time more for 5 neighbors over 1. 


Now let's use GoBGP as the MRT tester.
![gobgp reasonable tests](/assets/images/2021-08-bgp-stacks-internet/bgperf_gobgp-mrt-reasonable-internet_elapsed.png)

It's pretty clear for the majority of the stacks, the MRT tester (GoBGP) is the bottleneck. RustyBGP could not finish the 5 neighbors. I don't know why, I tried it at least 3 times. Kind of weird that OpenBGP at 5 neighbors is so much slower than all the others at 5 neighbors, not sure what that means when it is not with bgpdump2.


Let's go back to the previous blog post and bring back exabgp to compare against MRT. Remember that these are all unique /32s so aren't doing any testing of best path.
![exa reasonable tests](/assets/images/2021-08-bgp-stacks-internet/bgperf_exa-reasonable-800K_route_reception.png)

Not sure what this means. GoBGP is even slower than with bgpdump2. RustyBGP is faster? BIRD is slower with 5 neighbors ExaBGP than with bgpdump2.

# The setup

The setup to these tests matter a lot because they are really stressing these stacks. You might remember, that I'm using [bgperf](https://github.com/jopietsch/bgperf) to do the tests. bgperf has the idea of a target (the stack being tested), a monitor that connects to the target, is advertised all the routes, and counts how many have been received, and a set of testers. 

The bgpdump2 tests are again done on my AMD system with 32 cores and 64GB RAM. The tests with GoBGP as the generator were done on an EC2 m5.24.xlarge with 96 cores, and 384GB of RAM

The monitor is GoBGP. The role of the monitor is really interesting in these tests, specifically with bgpdump2. bgperf now checks to see on the target that each neighbor has sent all the expected # of prefixes and the test doesn't pass until both the monitor has received the expected number and has the target from each neighbor. In some tests, definitely with RustyBGP, I noticed that all the neighbors were done, but then much later would the monitor receive all the routes.

The way these works is that each of the testers is in a separate containers. All of the containers are started, without starting the MRT generator, and the each generator is started and there is a sleep between generators, just to be a little less severe to the target.

## MRT

As mentioned MRT is the standard way of recording internet routing tables. [Routeviews](http://www.routeviews.org/routeviews/) is a collection of people who voluntarily capture MRT data of the internet routes. You can grab their data. Inside one of these MRT files are the route tables of several different routers from many different ASes at the same time.

The specific MRT file used for these tests was [http://archive.routeviews.org/bgpdata/2021.08/RIBS/rib.20210801.0000.bz2](http://archive.routeviews.org/bgpdata/2021.08/RIBS/rib.20210801.0000.bz2)
## bgpdump2 as MRT generator

bgpdump2 is pretty much just an MRT playback cannon. Very fast.

I did find some interesting things when getting bgpdump2 to play back. When I first set it up, I had it sending from an arbitrary AS and no next-hop-self rewriting. This worked fine for FRR and RustyBGPd, but not BIRD or OpenBGPD. BIRD required both to work, OpenBGPD needed the correct AS to work, but not the next-hop-self. I don't know what any of that means, but I did find it.

## GoBGP as MRT generator

GoBGP uses a lot more resources which is why I ran it on the EC2 instance, so that the tests wouldn't run out of resources. It is much slower, under 10 neighbors it seems to be the bottleneck to most of these stacks. In other words, under 10 neighbors we aren't testing the BGP daemon, we are testing the playback.

# Lessons Learned
## RustyBGP
There are several issues with RustyBGP and these tests. First off, I couldn't get it to always finish. With bgpdump2, I got inconsistent results with 20 or more neighbors. Some neighbors never established connection. Sometimes all the neighbors received all the routes, but it never forwarded them all onto the monitor.

RustyBGP also blocks a lot when trying to get the neighbor data using the GoBGP client, even if there is available CPU on the machine. bgperf tries to get neighbor info every second, but also I sometimes run commands to get the data to see what is going on, and it will block for 10s of seconds.

The other big issue is that my results are not at all the same as the results from the creator of RustyBGP. [https://twitter.com/brewaddict/status/1427781197191475208](https://twitter.com/brewaddict/status/1427781197191475208). ![RustyBGP results from FUJITA](https://pbs.twimg.com/media/E9B8klXVgAI049H?format=jpg&name=large) Actually, the results for BIRD and OpenBGPD are similar to the GoBGP generator tests I have, but the RustyBGP are very different. There's a lot that are different in our testing. I think the biggest is that bgperf requires all the prefixes sent to a monitor and it checks that all the prefixes have been received by the target. His test checker update-watcher checks to see that the number of messages has stopped changing. In the cases I've seen in which the prefixes never got sent, this would show very different results than what I see. He is also using multiple VMs, including separate VMs for the testers. However, I made sure that the tests had idle CPU so the advantage of more machines shouldn't have matters. He is using GoBGP as a tester, as are some of the bgperf tests here, however he is using a custom one to be more efficient as a tester. I tried that out and got the same results, though it did use a little bit less memory.

Still, I don't know why my results are so very different.

## OpenBGPD
With bgpdump2, OpenBGPD would connect all 30 neighbors, run for a long time, get to where the target had received full prefixes (800k) from 29 of the 30 neighbors, and then the prefixes would drop to 0 for 2-3 neighbors and never increment. I ran this at least 3 times with the same results. I don't think the neighbors timed out. I do not know what was going on. Because bgpdump2 plays back so fast, I wonder how much this is artificial, but BIRD and FRR do fine with this. In the tests with GoBGP OpenBGPD doesn't have this problem.

I've started a conversation with OpenBGPD people, we'll see what we find. 
## bgperf

I'm still learning how to use bgperf usefully. It's clear that using exabgp as a tester does well for many neighbors, but it's too slow for many neighbors. Also because it's using unique /32s, it doesn't allow any best path comparison.

Using MRT playback through gobgp or bgpdump does much better for internet size tables

# Conclusions
Is bgpdump2 or GoBGP MRT playback more realistic? Don't know, but the different results are interesting. Are the GoBGP or bgpdump results better to examine? I don't know.

In these tests BIRD is the winner. That doesn't mean in all tests, just these. Is there enough difference between BIRD and FRR to not use FRR? Unlikely. Is there enough difference to not use OpenBGPD? Maybe because that's more like > 2x difference but only if you are really stressing your routing daemon. I mentioned that these are more realistic than my previous tests. That is definitely true for internet routes. However, if you have a RouteServer with > 1000 neighbors, they might all have unique prefixes, and so those tests might be realistic for that scenario.

Of course I'm still not doing any filtering, which might change performance characteristics.

There is a big mismatch between what I am seeing from RustyBGP and what [@brewaddict](https://twitter.com/brewaddict) is seeing. I'm not sure what that means.

 I'll say something possibly controversial. When I started I was hoping to find that there is a viable option that uses the resources of a modern machine and can converge much faster than even BIRD can in extreme situations like 50 neighbors with BGP routes. Waiting 300+ seconds for convergence seems like a very long time in 2021. Maybe when I get to testing commercial stacks I can find something faster.

# Questions
If anybody has other test results or other performance testing tools for BGP, I'd appreciate hearing about those. 