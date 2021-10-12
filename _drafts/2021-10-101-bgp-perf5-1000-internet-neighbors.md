---
layout: post
comments: true
author: Justin Pietsch
title: BGPerf 5 -- 1000 full internet neighbors
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
A major question is if it's reasonable to be able to test 1000 neighbors with full internet tables (800K prefixes). I wasn't sure, but [bgperf](https://github.com/jopietsch/bgperf) has support for playing back mrt files using [bgpdump2] which is very efficient. As always, I'm testing multiple things. The first is if bgperf on the hardware I have available usefully perform this test. The second is how well these stacks do with this many prefixes and neighbors. For the first I'm using different hardware: home computer with 64 GB RAM, EC2 M5.xlarge 24 with 96 cores and 384 GB RAM, and EC2 M5z with 48 cores and 184 GB RAM. The M

The baseline is 100 neighbors with 800K prefixes each.

![elapsed time](/assets/images/2021-10-bgp-5/bgperf_1M-100n_elapsed.png)

There's only data for FRR for the AMD because I was too lazy to run the other tests.

So this shows several things. I'll talk about the test system first. Each of these 3 platforms performed ok. for BIRD, the AMD is the fastest, then the M5z, then the M5, but not by that much. That makes sense, home CPU have faster single core performance, then the M5z, then the M5. I assume FRR would look the same if I had run all the tests. For rustybgp, it's the reverse, which is what we'd hope since it can take advantage of more cores

Lets' look at the performance of the stacks. First, BIRD, FRRouting and RustyBGP can all handle this load. Second, rustybgp is > 5 times faster. That's pretty amazing. FRR is a little bit faster than BIRD.

![min free Mem](/assets/images/2021-10-bgp-5/bgperf_1M-100n_min_free.png)

As long as it doesn't run out of memory, each of these platforms can do reasonable tests. I tried FRR at 200 neighbors and the AMD ran out of memory.

# FRR
 
 I've added some graphs to show how things go over time with any give test. FRR at 100 neighbors.

The number of prefixes received at the monitor.

![prefixes received at monitor](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_mon_received.png)

This graph shows the number of neighbors that the target has received all it's prefixes:

![neighbors full received routes](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_neighbors.png)

The CPU utilization of FRR

![cpu](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_cpu.png)


memory used by FRR
![memory used](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_mem_used.png)

The % idle of the machine. This shows that there are plenty of CPU resources available.

![% idle of machine](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_machine_idle.png)


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

# RustyBGP

RustyBGP finishes 200 neighbors just fine, but fails at 250. bgperf, every second, gets received prefix counts from the target. For rustybgp with 200 neighbors it stops returning that responding to the data request, and then bgperf just goes forever because it needs that data to know if anything has finished. I'm not sure what's going on with rustybgp, but I noticed this in previous posts that I can make it so that it doesn't respond to the management requestss.

![prefixes received at monitor](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_200_mon_received.png)

The number of neighbors that the target has received all it's prefixes:

![neighbors full received routes](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_200_neighbors.png)

The CPU utilization of the target

![cpu](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_200_cpu.png)


memory used by target
![memory used](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_200_mem_used.png)

The % idle of the machine. This shows that there are plenty of CPU resources available.

![% idle of machine](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_200_machine_idle.png)
# BIRD
BIRD succeeds at 200 neighbors, and fails by 225.  It does something similar to what FRR does in which it doesn't accept prefixes for every 