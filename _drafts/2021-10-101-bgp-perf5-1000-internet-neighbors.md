---
layout: post
comments: true
author: Justin Pietsch
title: BGPerf 5 -- 1000 full internet neighbors
excerpt: What happens if we try to get 1000 Neighbors with full internet routes? Is that realistic? I don't know
description: Testing to the limit of how many neighbors these BGP stacks can have with full internet routes
---


- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Follow-up Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)
- 4th Post [Bird on Bird, Episode 4 of BGP Perf testing ](https://elegantnetwork.github.io/posts/bird-on-bird-bgp-perf-episode4)

In the 3rd Post we compared full internet stacks with up to 50 neighbors. What happens if we try to get 1000 Neighbors with full internet routes? Is that realistic? I don't know. I have heard of people with 1M L3VPN routes that want hundreds of neighbors. I didn't think it would be useful, but then I started running experiments and found that I could break the stacks in interesting ways so I figured I should write up a blog post.


# Test setup
A major question is if it's reasonable to be able to test 1000 neighbors with full internet tables (800K prefixes). I wasn't sure, but [bgperf](https://github.com/jopietsch/bgperf) has support for playing back mrt files using [bgpdump2] which is very efficient. As always, I'm testing multiple things. The first is if bgperf on the hardware I have available usefully perform this test. The second is how well these stacks do with this many prefixes and neighbors. For the first I'm using different hardware: home computer with 64 GB RAM, EC2 M5.xlarge 24 with 96 cores and 384 GB RAM, and EC2 M5z with 48 cores and 184 GB RAM. The M

The baseline is 100 neighbors with 800K prefixes each.

![elapsed time](/assets/images/2021-10-bgp-5/bgperf_1M-100n_elapsed.png)

There's only data for FRR for the AMD because I was too lazy to run the other tests.

So this shows several things. I'll talk about the test system first. Each of these 3 platforms performed ok. for BIRD, the AMD is the fastest, then the M5z, then the M5, but not by that much. That makes sense, home CPU have faster single core performance, then the M5z, then the M5. I assume FRR would look the same if I had run all the tests. For RustyBGP, it's the reverse, which is what we'd hope since it can take advantage of more cores

Lets' look at the performance of the stacks. First, BIRD, FRRouting and RustyBGP can all handle this load. Second, rustybgp is > 5 times faster. That's pretty amazing. FRR is a little bit faster than BIRD.

![min free Mem](/assets/images/2021-10-bgp-5/bgperf_1M-100n_min_free.png)

As long as it doesn't run out of memory, each of these platforms can do reasonable tests. I tried FRR at 200 neighbors and the AMD ran out of memory.

## Detailed Baseline Graphs
 I've added some graphs to show how things go over time with any give test.

### Prefixes received at the monitor

FRR

![prefixes received at monitor](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_mon_received.png)

BIRD 

![prefixes received at monitor](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_100_mon_received.png)

RustyBGP

![prefixes received at monitor](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_100_mon_received.png)

RustyBGP curve looks a lot different. Don't know why.


### Neighbors that have sent the full table

FRRouting

![neighbors full received routes](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_neighbors.png)

BIRD

![neighbors full received routes](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_100_neighbors.png)

RusytBGP

![neighbors full received routes](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_100_neighbors.png)


RustyBGP looks different again. It's amazing that it can receive full routes for some neighbors within about 10 seconds. 


### Target CPU Utilization

FRR

![cpu](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_cpu.png)

BIRD

![cpu](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_100_cpu.png)

RustyBGP

![cpu](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_100_cpu.png)



### Memory used by target process

FRR

![memory used](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_mem_used.png)

BIRD

![memory used](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_100_mem_used.png)

RustyBGP

![memory used](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_100_mem_used.png)

Each of those looks different, don't know what it means.

### % idle of the machine.

FRR

![% idle of machine](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_machine_idle.png)

BIRD

![% idle of machine](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_100_machine_idle.png)

RustyBGP

![% idle of machine](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_100_machine_idle.png)

Some things to explain here. Notice the flat line at the beginning of each of these. That's a consequence of the way that bgperf is currently monitoring the host information. So we only have the last data point right before all the testers (bgpdump2) are fully running. It's a bug in the tester. Anyway, it does show, though, that when bgpdump first starts up it uses a lot of CPU resources, but after inital loading it barely uses anything. Even on the host with the least resources (AMD), there is plenty of CPU available on the host for the majority of the test. Except for RustyBGP, which uses most of the CPU.



# 1000 neighbors ?
## FRR



At 150 neighbors, FRR does not finish ever. Several problems. Some of the neighbors never send any prefixes successfully and the total prefixes received by the monitor doesn't seem to ever get to the amount required. I ran one test for over 6 hours and it never finished.


If we look at the target we can see the total number of connections:
```bash
$docker exec bgperf_frrouting_compiled_target vtysh -c 'sh ip bgp summ'|awk '{print $10}'|egrep -v '^$'|grep -v 'State'|wc
    152     152     985
```

152 is the right number. It's 150 tests + the monitor + 

But the interesting thing is that even hundreds of seconds into the test, many of the neighbors have still not sent prefixes successfully. Active state means the neighbor relationship is up and active, up there are no received prefixes
```bash
$ docker exec bgperf_frrouting_compiled_target vtysh -c 'sh ip bgp summ'|awk '{print $10}'|egrep -v '^$'|grep -v 'State'|grep 'Active'|wc
     42      42     294
```
So that's 40 neighbors that will never successfully send a prefix. I don't know why or what, I just know this is what happens. I saw the same thing with the other two hardware platforms.

This would be better if bgperf tracked the neighbors that have successfully sent any prefixes, but that's going to take more work.


![prefixes received at monitor](/assets/images/2021-10-bgp-5/frr_c_bgpdump2_800000_150_mon_received.png)

The number of neighbors that the target has received all it's prefixes:

![neighbors full received routes](/assets/images/2021-10-bgp-5/frr_c_bgpdump2_800000_150_neighbors.png)

The CPU utilization of the target

![cpu](/assets/images/2021-10-bgp-5/frr_c_bgpdump2_800000_150_cpu.png)


memory used by target
![memory used](/assets/images/2021-10-bgp-5/frr_c_bgpdump2_800000_150_mem_used.png)

The % idle of the machine. This shows that there are plenty of CPU resources available.

![% idle of machine](/assets/images/2021-10-bgp-5/frr_c_bgpdump2_800000_150_machine_idle.png)

I don't see anything interesting in these graphs.
# BIRD
BIRD succeeds at 200 neighbors, and fails at 300.  It gets to the place in which it does not increment either more prefixes at the monitor nor more neighbors finished for 120 seconds. So it just gets stuck, right around 215 neighbors.

```bash
bird,bird,v2.0.8-59-gf761be6b,300,800000,792000,878273,0,6975,3,6972,7148.89,102,27.503,85,91.046,-s,2021-10-08,48,184.71GB,0,FAILED,FAILED: stuck received count 878273 neighbors_checked 216
```

Can we see anything interesting in the benchmark graphs?

![prefixes received at monitor](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_300_mon_received.png)

![neighbors full received routes](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_300_neighbors.png)

![cpu](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_300_cpu.png)

![memory used](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_300_mem_used.png)

![% idle of machine](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_300_machine_idle.png)

## RustyBGP

Some of the tests I ran into a problem I mentioned in the previous blog post: every second bgperf collects information from the target about how many prefixes have been received by each neighbor. After a while, in some of the tests rustybgp stops responding to that data, which completely freaks out bgperf and it fails. I can't completely quantify when and why this happens, but it does and is a problem at least for testing.


monitor 

![prefixes received at monitor](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_500_mon_received.png)


neighbors

![neighbors full received routes](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_500_neighbors.png)


memory used by target
![memory used](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_500_mem_used.png)

The % idle of the machine. This shows that there are plenty of CPU resources available.

![% idle of machine](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_500_machine_idle.png)



# Conclusion
Probably, it's possible to measure these stacks with BGPerf at 1000 neighbors of internet routes. I ran out of memory at 384 GB. There are instances available with more memory than that. 

However, only RustyBGP can can near that level. And it requires a lot of memory. Both for RustyBGP and for the testers

FRR breaks first, at about 150 neighbors. Next is BIRD at between 200 and 250 neighbors.

We also added new graphs to see what these stacks are doing 