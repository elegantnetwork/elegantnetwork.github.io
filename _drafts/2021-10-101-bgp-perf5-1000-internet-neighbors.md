---
layout: post
comments: true
author: Justin Pietsch
title: BGP Performance 5 -- 1000 full internet neighbors
excerpt: What happens if we try to test with 1000 Neighbors with full internet routes to open source BGP stacks?
description: Testing to the limit of how many neighbors these BGP stacks can have with full internet routes
---


- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Follow-up Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)
- 4th Post [Bird on Bird, Episode 4 of BGP Perf testing ](https://elegantnetwork.github.io/posts/bird-on-bird-bgp-perf-episode4)

In the 3rd Post we compared these open source BGP stacks with neighbors that had full internet stacks with up to 50 neighbors. What happens if we try to get 1000 Neighbors with full internet routes? Is that realistic? I don't know. I have heard of people with 1M L3VPN routes that want hundreds of neighbors. I had not thought this sort of testing would be useful, but then I started running experiments and found that I could break the stacks in interesting ways so I figured I should write up a blog post.

# Test setup
A major question is if it's reasonable to be able to test 1000 neighbors with full internet tables (800K prefixes). I wasn't sure, but [bgperf](https://github.com/jopietsch/bgperf) has support for playing back mrt files using [bgpdump2](https://github.com/rtbrick/bgpdump2) which is very efficient. As always in these blog posts, I'm testing multiple things. The first is if bgperf on the hardware I have available usefully perform this test. The second is how well these stacks do with this many prefixes and neighbors. For the first I'm using different hardware: home computer with 64 GB RAM, EC2 M5.xlarge 24 with 96 cores and 384 GB RAM, and EC2 M5z with 48 cores and 184 GB RAM. I want to be able to take advantage of the cheapest hardware available for any test. It turns out the most important thing is not running out of memory.

# Comparing stacks

The baseline is 100 neighbors with 800K prefixes each.

![elapsed time](/assets/images/2021-10-bgp-5/bgperf_1M-100n_elapsed.png)

I didn't do complete testing for FRRouting and OpenBGPD because I think you can see the general trends.

These results show several things. I'll talk about the test system first. Each of these 3 platforms performed fine. For BIRD, the AMD is the fastest, then the M5z, then the M5, but not by that much. That makes sense, home CPU have higher frequency single core, , then the M5z, then the M5. I assume FRRouting and OpenBGPD would look the same if I had run all the tests. For RustyBGP, it's the reverse, which is what we'd hope since it can take advantage of more cores

Let's look at the performance of the stacks. First, BIRD, FRRouting and RustyBGP can all handle this load. Second, RustyBGP is > 5 times faster. That's pretty amazing. FRRouting is a little bit faster than BIRD.

![memory usage](/assets/images/2021-10-bgp-5/bgperf_1M-100n_max_mem.png)

OpenBGPD still uses the most memory as we've discussed in the past.

![min free Mem](/assets/images/2021-10-bgp-5/bgperf_1M-100n_min_free.png)

As long as it doesn't run out of memory, each of these host platforms can do reasonable tests. I tried FRRouting at 200 neighbors and the AMD ran out of memory.

## On the M5z
Let's see how each of these scales as they go from 50 to 200 neighbors.


![elapsed time](/assets/images/2021-10-bgp-5/bgperf_M5z_elapsed.png)

![memory usage](/assets/images/2021-10-bgp-5/bgperf_M5z_max_mem.png)
## Detailed Baseline Graphs on the M5z
 I've added some graphs to show how things go over time with any give test. For each individual test, bgperf now produces graphs. I'm not sure of the use of these graphs, but we'll try them out here.

### Prefixes received at the monitor

FRRouting

![prefixes received at monitor](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_mon_received.png)

BIRD 

![prefixes received at monitor](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_100_mon_received.png)

OpenBGPD

![prefixes received at monitor](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_100_mon_received.png)

RustyBGP

![prefixes received at monitor](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_100_mon_received.png)

RustyBGP curve looks a lot different. Don't know why. On the other hand, RustyBGP completes in 160 seconds, while OpenBGPD completes in 1407s, almost 9 times faster.


### Neighbors that have sent the full table

FRRouting

![neighbors full received routes](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_neighbors.png)

BIRD

![neighbors full received routes](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_100_neighbors.png)

OpenBGPD

![neighbors full received routes](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_100_neighbors.png)

RusytBGP

![neighbors full received routes](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_100_neighbors.png)


It's interesting how OpenBGP looks compared to BIRD and FRR. None of it's neighbors complete until almost at the end, while FRRouting and BIRD start completing much sooner. While the others show either neighbors completing before the monitor gets to 800K, or the monitor gets to 800K before the neighbors, OpenBGPD has them pretty close to in sync.

RustyBGP looks different again. It's amazing that it can receive full routes for some neighbors within about 10 seconds. 


### Target CPU Utilization

FRR

![cpu](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_cpu.png)

BIRD

![cpu](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_100_cpu.png)

OpenBGPD

![cpu](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_100_cpu.png)

RustyBGP

![cpu](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_100_cpu.png)


These are pretty much as expected since the first three can only take advantage of the single core. Not sure what RustyBGP does at about 60 seconds that makes the utilization go up even more. Oh, wait, if we look below at the memory allocation, that lines up with when RustyBGP dramatically slows down it's allocation of memory. Oh, if we look at the percent idle, that's also when all the bgpdump2s are finished launching. Interesting. I'm not sure what it means, but there is some correlation there.

### Memory used by target process

FRR

![memory used](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_mem_used.png)

BIRD

![memory used](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_100_mem_used.png)

OpenBGPD

![memory used](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_100_mem_used.png)

RustyBGP

![memory used](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_100_mem_used.png)

Each of those looks different, don't know what it means, especially in comparing between the stacks. They do allocate differently.

### % idle of the machine.

FRR

![% idle of machine](/assets/images/2021-10-bgp-5/frr_c_bgpdump2__800000_100_machine_idle.png)

BIRD

![% idle of machine](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_100_machine_idle.png)

OpenBGPD

![% idle of machine](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_100_machine_idle.png)

RustyBGP

![% idle of machine](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_100_machine_idle.png)

Some things to explain here. Notice the flat line at the beginning of each of these. That's a consequence of the way that bgperf is currently monitoring the host information. So we only have the last data point right before all the testers (bgpdump2) are fully running. It's a bug in the tester. Anyway, it does show, though, that when bgpdump first starts up it uses a lot of CPU resources, but after inital loading it barely uses anything. Even on the host with the least resources (AMD), there is plenty of CPU available on the host for the majority of the test. Except for RustyBGP, which uses most of the CPU.



# 1000 neighbors ?
## FRR

At 150 neighbors, FRRouting does not finish ever. Several problems. Some of the neighbors never send any prefixes successfully and the total prefixes received by the monitor doesn't seem to ever get to the amount required. I ran one test for over 6 hours and it never finished.

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

prefixes received at the monitor:

![prefixes received at monitor](/assets/images/2021-10-bgp-5/frr_c_bgpdump2_800000_150_mon_received.png)

number of neighbors that the target has received all it's prefixes:

![neighbors full received routes](/assets/images/2021-10-bgp-5/frr_c_bgpdump2_800000_150_neighbors.png)

CPU utilization of the target:

![cpu](/assets/images/2021-10-bgp-5/frr_c_bgpdump2_800000_150_cpu.png)

memory used by target:

![memory used](/assets/images/2021-10-bgp-5/frr_c_bgpdump2_800000_150_mem_used.png)

% idle of the machine. This shows that there are plenty of CPU resources available.

![% idle of machine](/assets/images/2021-10-bgp-5/frr_c_bgpdump2_800000_150_machine_idle.png)

I don't see anything interesting in these graphs.
## BIRD
BIRD succeeds at 200 neighbors, and fails at 300.  Well, it gets stuck not incrementing either the prefixes received at the monitor or the number of neighbors completed. I usually have a 120s timeout, but I changed it to one hour. As you can see on the graphs, it got stuck for almost an hour at about 225 neighbors. I don't know what it was doing at that point in time. The total time was 15781s, more than 4 hours. That doesn't really count as success. It gets to about 225 neighbors in less than half that time. 

prefixes received at the monitor:

![prefixes received at monitor](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_300_mon_received.png)

number of neighbors that the target has received all it's prefixes:

![neighbors full received routes](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_300_neighbors.png)

CPU utilization of the target:

![cpu](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_300_cpu.png)

memory used by target:

![memory used](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_300_mem_used.png)

% idle of the machine:

![% idle of machine](/assets/images/2021-10-bgp-5/bird_bgpdump2_800000_300_machine_idle.png)

Other than the neighbor graph showing it getting stuck for almost an hour, nothing much interesting going on here.

## BIRD separate table per neighbor
BIRD in which each neighbor has it's own table cannot even do 100 neighbors. It gets to 88 neighbors and then stops. The CPU utilization drops to 0. Not sure what is going on between BIRD and bgdpdump2.

prefixes received at the monitor:

![prefixes received at monitor](/assets/images/2021-10-bgp-5/bird-no-s_bgpdump2_800000_100_mon_received.png)

number of neighbors that the target has received all it's prefixes:

![neighbors full received routes](/assets/images/2021-10-bgp-5/bird-no-s_bgpdump2_800000_100_neighbors.png)

CPU utilization of the target:

![cpu](/assets/images/2021-10-bgp-5/bird-no-s_bgpdump2_800000_100_cpu.png)

memory used by target:

![memory used](/assets/images/2021-10-bgp-5/bird-no-s_bgpdump2_800000_100_mem_used.png)

% idle of the machine:

![% idle of machine](/assets/images/2021-10-bgp-5/bird-no-s_bgpdump2_800000_100_machine_idle.png)
## OpenBGPD
OpenBGPD at 300 finishes, but takes so long that I'm not sure that should really count. Well, I didn't let it finish, it was going to ran out of memory on the machine I was using, it was past my bedtime and I didn't want to get charged for a machine over night.

prefixes received at the monitor:

![prefixes received at monitor](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_300_mon_received.png)

number of neighbors that the target has received all it's prefixes:

![neighbors full received routes](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_300_neighbors.png)

CPU utilization of the target:

![cpu](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_300_cpu.png)

![memory used](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_300_mem_used.png)

memory used by target:

![machine free mem](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_300_free_mem.png)

% idle of the machine:

![% idle of machine](/assets/images/2021-10-bgp-5/openbgp_bgpdump2_800000_300_machine_idle.png)

I don't see anything interesting in these other than in comparing to the other stacks, which we did above.
## RustyBGP

I successfully tested RustyBGP with 500 neighbors on the M5.24xlarge which has 96 cores and 384 GB of RAM. More neighbors than that ran into a problem I mentioned in the previous blog post: every second bgperf collects information from the target about how many prefixes have been received by each neighbor. After a while, in some of the tests RustyBGP stops responding to that data, which completely freaks out bgperf and it fails. I don't know if the problem is that there isn't enough CPU resources or something else. In other words, if RustyBGP had it's own dedicated resource and bgpdump2 it's own dedicated resources would this break? I can't be sure but I kind of think so since bgpdump2 does it's greatest work at the beginning when it's starting up.

at 500, it finishes in about 1700 seconds, which is not that much longer than the other three took to do 100 neighbors.

prefixes received at the monitor:

![prefixes received at monitor](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_500_mon_received.png)

number of neighbors that the target has received all it's prefixes:

![neighbors full received routes](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_500_neighbors.png)


memory used by target
![memory used](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_500_mem_used.png)

The % idle of the machine.

![% idle of machine](/assets/images/2021-10-bgp-5/rustybgp_bgpdump2_800000_500_machine_idle.png)



# Conclusion
Is it reasonable to test 1000 neighbors with full internet routes using bgperf? Possibly, though I didn't get past 500 neighbors. It is likely that 1000 neighbors would have needed more memory, and that is available on other EC2 instances which I'm not sure that there are other instances that have more CPU. At some point it would be necessary to figure out mutli-host bgperf. So I think it would work, but I couldn't prove it here. Each of these host platforms performed fine and there isn't much difference in performance unless you run out of memory. 

However, only RustyBGP can get near that level. And it requires a lot of memory. Both for RustyBGP and for the testers

FRRouting breaks first, at about 150 neighbors. OpenBGPD completes at 300 neighbors but it takes so long I don't think it's practical. Next is BIRD at between 200 and 250 neighbors, let's say 225. 

We also added new graphs to see what these stacks are doing 


How did each of these scale