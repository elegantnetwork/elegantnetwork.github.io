---
layout: post
comments: true
author: Justin Pietsch
title: Followup Measuring BGP Stacks
excerpt: 
description: Measuring
---

After I published [the post on measuring open source BGP stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/), I was embarrassed after I realized how haphazard the testing was. I was not very systematic about the way I tried different test parameters.  So I hacked on [bgperf](https://github.com/jopietsch/bgperf), added some more reporting and created a new batch feature that iterates through parameters systematically, and automatically graphs the results.

By request, I added [rustybgp](https://github.com/osrg/rustybgp) and [OpenBGPD](http://www.openbgpd.org/). I've only added rudimentary support for these just to get started testing the number of prefixes and neighbors. Rustybgp isn't yet a fully formed BGP stack; I don't think it has much support for any kind of policy yet. 

In these tests I stopped testing GoBGP, even though bgperf still supports it. As you can see from the previous post, it uses a lot more resources and is much slower. It makes the graphs much harder to interpret. I also only test the single table version of bird, because it's the easier config and it's faster.

Last time for FRR I (accidentally) used a dev version which took an excess amount of time establishing neighbors when there were > 30 neighbors. This time I'm using FRRouting 7.5.1 from dockerhub container and FRRouting 8.0 that bgperf compiled from github, neither of which have problems with neighbor times. I'm not going to show neighbor results here because for the most part they are uninteresting.

There are all kinds of different uses for BGP stacks. I try to cover a set of interesting scenarios to demonstrate how these perform.

The goal in all of this is to add some quantitative power to any decision making about different BGP stacks. It's not the only aspect to consider, often it's the least important.
# Results
The first set of results we'll look at are from my AMD 3950 32 core machine with 64 GB RAM.
## 10K prefixes
We'll start with 10K prefixes. It iterates through 10, 30, 50, and then 100 neighbors. It looks like 100 neighbors with each 10K prefixes is pretty taxing on many of these stacks.

Route reception is the time from the first route received until all routes are received.

![route reception time](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_10K_route_reception.png)


The first thing that jumps out is how much slower OpenBGPD is especially at 100, and even 50, neighbors. While harder to see, bird and rusytbgp at 100 neighbors are considerably slower than the two FRRoutings 18s compared to 9/10s.

![max cpu](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_10K_max_cpu.png)

Rustybgp uses a lot more  CPU resources, even though it isn't any faster than FRR or BIRD.

![max mem](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_10K_max_mem.png)

OpenBGPD uses a tremendous amount of memory, especially at 100 neighbors. I'm not sure what's going on here, but that looks troubling to me.

### results table
<script src="https://gist.github.com/jopietsch/b2a921ce33f06fa011c71a2d55c9c685.js"></script>

## Many (many) neighbors, 10 prefixes
This iterates through neighbors from 250 to 1750. More than that runs out of memory on my 64 GB machine (because of all the ExaBGP). It uses a very minimal amount of prefixes just to see if we can understand anything regarding max prefixes.

![route reception time](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_many_neighbors_10p_route_reception.png)

The time to receive prefixes, all 10 per neighbor, doesn't matter in this test.

![max cpu](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_many_neighbors_10p_max_cpu.png)

Interestingly, Rustybgp uses the smallest amount of CPU and OpenBGPD uses the most, by a lot. FRRouting 8 uses more  CPU at 1750 neighbors 7.5.1. I don't know that is important since it's still not very much

![max mem](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_many_neighbors_10p_max_mem.png)

OpenBGPD uses a lot more memory, several orders of magnitude more. The weirdest thing is that FRRouting 8 uses a lot more memory than FRRouting 7.5.1 or BIRD. Don't know if this matters because it's only 10 prefixes, but it's interesting.

### results table

<script src="https://gist.github.com/jopietsch/7fec4c43104b7bfa9295a0fd2697a742.js"></script>

## Many neighbors, 100 prefixes

This only goes from 250 to 750 neighbors with 100 prefixes. More than that runs out of memory on my 64 GB machine (because of all the ExaBGP)

![route reception time](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_many_neighbors_100p_route_reception.png)

Again, route reception is trivial. Maybe it matters that OpenBGPD at 750 neighbors is 4 seconds, but not sure.

![max cpu](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_many_neighbors_100p_max_cpu.png)

Similar to the 10 prefix tests, OpenBGPD uses many more CPU resources than the others.

![max mem](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_many_neighbors_100p_max_mem.png)

Similar to the 10 prefix tests, OpenBGPD uses many more memory resources than the others.

### results table


## what happens as prefixes grow significantly

This set of tests iterates from 50 to 250 neighbors and 10 prefixes to 1000.

![route reception time](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_prefix_growth_route_reception.png)

route reception is trivial, except OpenBGPD at 250 neighbors, 1000 prefixes. It plain freaks out.


![max cpu](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_prefix_growth_max_cpu.png)


![max mem](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_prefix_growth_max_mem.png)

### results table
<script src="https://gist.github.com/jopietsch/918047845cc0e5f84890cd7b0c175125.js"></script>

## 1M routes.
 Since the internet is getting close to 1M routes, I wanted to see how how these do with 1M routes and multiple neighbors

![route reception time](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_1M_route_reception.png)

This is the first time that RustyBGP is considerably slower than all the others


![max cpu](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_1M_max_cpu.png)


![max mem](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_1M_max_mem.png)

### results table
<script src="https://gist.github.com/jopietsch/191af811bd28b5b34546fb14377edac1.js"></script>


# tests on EC2 m5 

I wanted to see what would happen with more resources, specifically twice as much RAM. However, the AMD device is a consumer device with less cores but each higher speed. That might matter to some of these tests because of the single core processes.

I won't go through all the graphs, because there are pretty similiar results. But where there is something interesting I'll mention that.


## 10K prefixes
We'll start with 10K prefixes. It iterates through 10, 30, 50, and then 100 neighbors. It looks like 100 neighbors with each 10K prefixes is pretty taxing on many of these stacks.

![route reception time](/assets/images/2021-08-followup-bgp-stacks/ec2-m5.12xlarge/bgperf_10K_route_reception.png)

Route Reception has a greater difference between both FRRs and BIRD/Rustybgp. Maybe it's that 3950 is a consumer device with higher speed cores?


### results table
<script src="https://gist.github.com/jopietsch/e146b31e87df3fadc0c122c6766d9f1d.js"></script>

## Many (many) neighbors, 10 prefixes

I didn't find any interesting differences here.

### results table



## Many neighbors, 100 prefixes

![route reception time](/assets/images/2021-08-followup-bgp-stacks/ec2-m5.12xlarge/bgperf_many_neighbors_100p_route_reception.png)

OpenBGPD at 750 neighbors is a lot greater route reception time than on the AMD.

### results table
## what happens as prefixes grow significantly

Because of more memory, I tested to 500 neighbors.

![route reception time](/assets/images/2021-08-followup-bgp-stacks/ec2-m5.12xlarge/bgperf_prefix_growth_route_reception.png)

OpenBGPD is an order of magnitude worse at 500 neighbors, 1000 prefixes, then at 250.

![max cpu](/assets/images/2021-08-followup-bgp-stacks/ec2-m5.12xlarge/bgperf_prefix_growth_max_cpu.png)

We see Rusytbgp jump up in CUP usage at 500 neighbors

![max mem](/assets/images/2021-08-followup-bgp-stacks/ec2-m5.12xlarge/bgperf_prefix_growth_max_mem.png)

OpenBGPD is using over 30GB of RAM and is slow at 500 neighbors.
### results table


## 1M routes.
 Since the internet is getting close to 1M routes, I wanted to see how how these do with 1M routes and multiple neighbors


![max cpu](/assets/images/2021-08-followup-bgp-stacks/ec2-m5.12xlarge/bgperf_1M_max_cpu.png)

RustyBGP uses twice as much CPU as the AMD, but is still slower than all the others.

### results table


# results for EC2 t3

I wanted to try out what would happen on a machine with more limited resources. The biggest issue is that because the target stack and the tester, and the monitor, are all on the same hardware they fight for resources, especially in the many neighbors test. I'm still trying to understand the impact of bgperf on the results.

## 10K prefixes

![max cpu](/assets/images/2021-08-followup-bgp-stacks/ec2-t3.2xlarge/bgperf_10K_max_cpu.png)

It's interesting that RustyBGP uses less CPU, because there is less, but it's not much slower than on the other machines. What is it doing with those other CPU resources?

### results table


## Many (many) neighbors, 10 prefixes
Can only test many less parameters because of memory.


![max cpu](/assets/images/2021-08-followup-bgp-stacks/ec2-t3.2xlarge/bgperf_many_neighbors_10p_max_cpu.png)

While not really different, it is interesting to zoom in on just 250 and 500 neighbors.

![max mem](/assets/images/2021-08-followup-bgp-stacks/ec2-t3.2xlarge/bgperf_many_neighbors_10p_max_mem.png)

Again, not much different, but zooming in shows the difference in memory between FRRouing 8/OpenBGPD vs the others.

### results table


## Many neighbors, 100 prefixes

Nothing interestingly different here.

### results table


## what happens as prefixes grow significantly

Nothing interestingly different here.

### results table


## 1M routes.


![route reception time](/assets/images/2021-08-followup-bgp-stacks/ec2-t3.2xlarge/bgperf_1M_route_reception.png)

While the others are about the same, Rustybgp doubles it's time here.

![max cpu](/assets/images/2021-08-followup-bgp-stacks/ec2-t3.2xlarge/bgperf_1M_max_cpu.png)

Just a lot less CPU resources for RustyBGP to use.

### results table



# Conclusion

OpenBGP struggles in places that the other stacks do not, but it's still faster than GoBGP. For anybody running OpenBGP, are there scenarios that make it more useful than the other stacks?

Run bgperf yourself


Are the scenarios that I'm using to test useful? Are there different combinations of prefixes and neighbors that I should be testing?

Does anybody else have either BGP testing tools that they can share, or other results that they can share?

# followup / todo

It would be interesting to test these on even more constrained devices. However, I need to figure out a way to have the target stacks isolated and not consumed by the tester ExaBGP processes.