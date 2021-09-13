---
layout: post
comments: true
author: Justin Pietsch
title: Bird on Bird, Episode 4 of BGP Perf testing 
excerpt: 
description: But as we'll see, it shows interesting results.
---

- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Followup Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)
- 4th Post [Bird on Bird, Episode 4 of BGP Perf testing ](https://elegantnetwork.github.io/posts/bird-on-bird-bgp-perf-episode4)

After the last post, I thought the this post would be either adding interesting BGP policy. But that's tricky. It's going to take some rethinking about how bgperf measures when a test is done, figuring out what filter is useful is hard, and also I want a little bit of a multi-vendor approach because there are already 5 NOSes and I want to add more. In the meantime, 
Maria Matejka <maria.matejka@nic.cz> added a BIRD generator instead of ExaBGP because BIRD is faster and uses less memory. Also, Fujita Tomonori updated RustyBGP based on the results I found and improved it's performance. In seeing the affect of each of these I discovered some things I'm surprised about.

These tests use the same versions of NOSes as before except for RustyBGP. It's using the latest as of 2021-09-11.

So I tried using BIRD as a generator instead of Exa. In the process I've added a little more sophistication to bgperf. Now if it sees the amount of prefixes received at the monitor and the number of neighbors complete hasn't changed for 30 intervals (seconds) it fails the test. Also if the number of prefixes received at the monitor drops for more than 5 intervals then it fails. 

Last time I noticed that with Bgpdump2 as the tester, OpenBGPD as the target and 30 neighbors, OpenBGPD stalled and then dropped a neighbor. I still don't know why, but I've added a new errors column in the stats. For BIRD and Bgpdump2 as testers, it will grep the tester logs to see if there are errors and count them.

I'm starting to get to the place where I find weird things, maybe because of the test infrasture, maybe because of specific NOS behavior, and adapting and figuring out what's going on takes extra time.

Let's get to the data

# Unique prefixes using BIRD and ExaBGP as terter
## BIRD as tester / generator with many many neighbors
This is a test that is harder than I did with ExaBGP. I couldn't get Exa to do 1000 prefixes for this many neighbors. This might not be a realistic set of tests. I don't know if people have 1000 prefixes from 1500 neighbors. But as we'll see, it shows interesting results.

If there isn't an entry for a NOS, it's because the test failed for one of the reasons above: 30 seconds stuck at the same number of prefixes received at the monitor or the number of prefixes dropped for 5 seconds.

![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_90hold_elapsed.png)

Not what I expected at all. BIRD is considerably worse than everything else, because I forgot about previous results. Looking back [the second post](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/) in the section about extreme tests, BIRD does much worse than everybody else at 1000 neighbors, 10 prefixes, or 500+ neighbors, 1000 prefixes. That's what we are seeing here as well. I had just forgotten those results. It's just that with BIRD as a tester it's faster and easier to test these extremes.

Though it's hard to see in this graph because of BIRD's outsized times, but RustyBGP looks like it got better than it was before. It's still slower than FRR, but not 10x slower.

![memory usage](/assets/images/2021-09-bgp-episode4/bgperf_90hold_max_mem.png)

BIRD is using a lot more memory than we've see it. Actually, I think something weird is going on in the way bgperf is measuring memory in this case that I haven't seen before.

![free memory](/assets/images/2021-09-bgp-episode4/bgperf_90hold_min_free.png)

This graphs shows the minimum free on the 64 GB machine I'm using for this test. with BIRD there's lots of free memory. Whereas with OpenBGPD and RustyBGP the min free lines up with the amount free. I'm not sure what's going on here, it's weird.

![tester errors](/assets/images/2021-09-bgp-episode4/bgperf_90hold_tester_errors.png)


<script src="https://gist.github.com/jopietsch/d3e6ad7d946229d2bb967613012529b6.js"></script>
### TODO many neighbors, 10 prefixes

## ExaBGP many neighbors

Just to make sure that the other changes made don't give us different results, also ran similar tests using exaBGP as the tester.

### RustyBGP
I cannot get RustyBGP to work with ExaBGP as the tester. I just get
```
$ more /tmp/bgperf/rustybgp/rustybgp.log
Hello, RustyBGPd (32 cpus)!
10.10.0.2: down std::io::Error
10.10.0.3: notification 5 2
```
I think that means a BGP Error (Notification) code of 5,2, which would be FSM, Receive Unexpected Message in OpenConfirm State.

I did no debugging. Exa works with all the other BGP stacks just fine, and RustyBGP works with the other testers, so I don't know what's going on.

## Bird generator, 1M prefixes
![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_1M_bird_elapsed.png)
![memory usage](/assets/images/2021-09-bgp-episode4/bgperf_1M_bird_max_mem.png)

![max cpu](/assets/images/2021-09-bgp-episode4/bgperf_1M_bird_max_cpu.png)



## Exa generator, 1M prefixes

![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_1M_exa_elapsed.png)
![memory usage](/assets/images/2021-09-bgp-episode4/bgperf_1M_exa_max_mem.png)

![max cpu](/assets/images/2021-09-bgp-episode4/bgperf_1M_exa_max_cpu.png)


# MRT
## Bbgpdump2 again

TO see if RustyBGP can do better than it did before. These results might not be exactly the same as before because bgperf now checks to see if things are stalled or prefixes count is dropping.

![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_bgpdump_elapsed.png)

Yep, RustyBGP is considerably better than before and now the fastest in this test. 

Why do OpenBGP and BIRD have less results than before? It's because it has had test failures.

![memory usage](/assets/images/2021-09-bgp-episode4/bgperf_bgpdump_max_mem.png)

![free memory](/assets/images/2021-09-bgp-episode4/bgperf_bgpdump_min_free.png)

![max cpu](/assets/images/2021-09-bgp-episode4/bgperf_bgpdump_max_cpu.png)

Finally we are seeing what I'd hoped to see when I started this, a BGP stack that used more CPU resources and was faster than other.woot woot.


![tester errors](/assets/images/2021-09-bgp-episode4/bgperf_bgpdump_tester_error.png)

Why aren't there any tester errors? That's because any test that FAILED, we don't put in graphs. But if we look at the data directly for all the FAILED, we can see that for all the openbgp fails, it did notice errors (which is the last column before the FAIL.)

```
$ grep FAIL results/bgpdump.csv
openbgp,openbgp,OpenBGPD 7.1,30,800000,800000,877885,0,270,3,267,292.84,192,11.266,74,41.129,,2021-09-11,32,62.82GB,1,FAILED,FAILED: stuck received count 877885 neighbors_checked 29
openbgp,openbgp,OpenBGPD 7.1,40,800000,800000,878406,0,412,3,409,441.48,164,14.638,74,35.855,,2021-09-11,32,62.82GB,2,FAILED,FAILED: stuck received count 878406 neighbors_checked 39
bird,bird,v2.0.8-59-gf761be6b,50,800000,792000,871147,7,190,3,187,233.67,102,4.162,76,44.243,-s,2021-09-11,32,62.82GB,0,FAILED,FAILED: dropping received count 871147 neighbors_checked 11
openbgp,openbgp,OpenBGPD 7.1,50,800000,800000,878406,0,570,3,567,606.06,171,18.505,77,29.774,,2021-09-11,32,62.82GB,2,FAILED,FAILED: stuck received count 878406 neighbors_checked 49
```


<script src="https://gist.github.com/jopietsch/c4bff9381f5c93b68c20b3f794f43072.js"></script>


## GoBGP MRT
![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_gobgp-mrt_elapsed.png)

![memory usage](/assets/images/2021-09-bgp-episode4/bgperf_gobgp-mrt_max_mem.png)

![free memory](/assets/images/2021-09-bgp-episode4/bgperf_gobgp-mrt_min_free.png)

![max cpu](/assets/images/2021-09-bgp-episode4/bgperf_gobgp-mrt_max_cpu.png)


```
$ grep FAILED gobgp-mrt.csv
frr,frr,FRRouting 7.5.1_git (6d71b511efd5).,20,800000,744000,796303,1,365,5,360,380.56,214,4.408,11,206.953,,2021-09-13,64,249.02GB,0,FAILE
,FAILED: dropping received count 796303 neighbors_checked 19
frr 8,frr_c,FRRouting 8.0-bgperf (136036a6020d).,20,800000,744000,796289,1,306,7,299,321.15,219,4.187,10,204.979,,2021-09-13,64,249.02GB,0,FAILED,FAILED: dropping received count 796289 neighbors_checked 14
openbgp,openbgp,OpenBGPD 7.1,20,800000,744000,796306,0,1006,8,998,1020.5,212,16.343,2,178.892,,2021-09-13,64,249.02GB,0,FAILED,FAILED: stuck received count 796306 neighbors_checked 0
rustybgp,rustybgp,rustybgpd,20,800000,744000,796719,7,330,7,323,351.75,651,12.371,7,200.624,,2021-09-13,64,249.02GB,0,FAILED,FAILED: stuck received count 796719 neighbors_checked 13
frr,frr,FRRouting 7.5.1_git (afed60012ff9).,30,800000,744000,796304,0,571,5,566,590.93,214,6.467,4,190.913,,2021-09-13,64,249.02GB,0,FAILED
FAILED: dropping received count 796304 neighbors_checked 12
frr 8,frr_c,FRRouting 8.0-bgperf (df1d18a186e3).,30,800000,744000,796288,1,491,5,486,512.13,216,6.521,3,187.757,,2021-09-13,64,249.02GB,0,FAILED,FAILED: dropping received count 796288 neighbors_checked 13
openbgp,openbgp,OpenBGPD 7.1,30,800000,744000,796305,0,1412,6,1406,1431.76,217,23.448,1,152.019,,2021-09-13,64,249.02GB,0,FAILED,FAILED: stuck received count 796305 neighbors_checked 0
rustybgp,rustybgp,rustybgpd,30,800000,744000,245197,15,385,6,379,420.65,681,13.594,2,182.888,,2021-09-13,64,249.02GB,0,FAILED,FAILED: stuck received count 245197 neighbors_checked 7
```

# Hold Timers

What should BGP Hold Timer be for a test like this? The default is 90 seconds, but if the test doesn't even take 90 seconds then we aren't testing if hold timers will work.

### Hold Timers 5 on BIRD


## What did we learn
I thought I'd learned about a BIRD performance issue, but it turns out I already knew that. Also, are the tests that BIRD did bad on realistic? Unlikely. It's finding an edge case in which BIRD performs badly, but you are unlikely to get to that place.

BIRD as a tester is a tester that takes less resources, so I've made it the default tester now.

Did RustyBGP get better? Yes, much better. It's the fastests in the MRT playback tests. It's not as fast as FRR in the many neighbors tests, but those are less realistic.

# Stuff
[bgperf](github -- bgperf)