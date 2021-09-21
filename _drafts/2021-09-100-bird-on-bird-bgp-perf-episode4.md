---
layout: post
comments: true
author: Justin Pietsch
title: Bird on Bird, Episode 4 of BGP Perf testing 
excerpt: these posts are as much about me figuring out how to test BGP stacks as they are about actually measuring those stacks
description: This post dives into specific issues with using BIRD as a tester and testing RustyBGP improvements.
---

- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Followup Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)
- 4th Post [Bird on Bird, Episode 4 of BGP Perf testing ](https://elegantnetwork.github.io/posts/bird-on-bird-bgp-perf-episode4)

After the last post, I thought the this post would be either adding interesting BGP policy. But that's tricky. It's going to take some rethinking about how [bgperf](https://github.com/jopietsch/bgperf) measures when a test is done, figuring out what filter is useful is hard, and also I want a little bit of a multi-vendor approach because there are already 5 NOSes and I want to add more. In the meantime, Maria Matejka <maria.matejka@nic.cz> added a BIRD generator in place of ExaBGP because BIRD is faster and uses less memory. Also, Fujita Tomonori updated RustyBGP based on the results I found and improved it's performance. In seeing the affect of each of these I discovered some things I'm surprised about.

Remember, these posts are as much about me figuring out how to test BGP stacks as they are about actually measuring those stacks. This episode will talk about some of the changes that I've made and what they mean. The test infrastructure influences the results more than I might like. I think that's mostly okay because mostly the results are about comparing between stacks so the differences should still pop out. That's not always going to be true; sometimes differences will be masked.

These tests use the same versions of NOSes as before except for RustyBGP. It's using the latest as of 2021-09-11.

I tried using BIRD as a generator instead of Exa. In the process I've added a little more sophistication to bgperf. Now if it sees the amount of prefixes received at the monitor and the number of neighbors complete hasn't changed for 30 intervals (seconds) it fails the test. Also if the number of prefixes received at the monitor drops for more than 5 intervals then it fails. 


I'm starting to get to the place where I find weird things, maybe because of the test infrastructure, maybe because of specific BGP stack behavior, and adapting and figuring out what's going on takes extra time.

Let's get to the data

# Unique prefixes using BIRD and ExaBGP as tester

If there isn't an entry for a BGP stack, it's because the test failed for one of the reasons above: 30 seconds stuck at the same number of prefixes received at the monitor or the number of prefixes dropped for 5 seconds.

I know that some aspects of the test mechanism shows

## 10 prefixes

### BIRD as tester

![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_10_bird_elapsed.png)

This isn't what I expected. I don't remember BIRD doing so poorly. Something is going on when BIRD has many neighbors.

### ExaBGP as tester


![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_10_exa_elapsed.png)

These aren't the results I remember, let's actually look at what we saw before.

![elapsed time](/assets/images/2021-08-followup-bgp-stacks/AMD-3950/bgperf_many_neighbors_10p_route_reception.png)

These are the results from the 2nd post. Why is it so different than the Exa results bgperf creates now? It's about what to measure measuring and when things are started. There are three things in bgperf: tester, monitor, and target. Each has to start up, connect to the right neighbors, and then the tester needs to send all the routing updates. Originally bgperf started the tester, started the target, started the monitor, waited for the monitor to connect, then started the elapsed timer. This masks a bunch of data that the tester has already sent to the target. Also, bgpdump2 as tester doesn't work if you start the tester before the target. So I changed the order of things and now in the elapsed there is some amount of tester startup time, but during that time the tester is sending traffic to the target.

Because of the way bgperf was trying to exclude the tester time we masked a problem with BIRD. bummer.

Now bgperf includes tester time in it, because the tester is sending traffic to the traffic. The faster the tester startup the better.

Is this current method better? I think so. Good enough? I hope so. It would be better if we established all the BGP sessions and then just started all the sending at the same time. There might be a way to do that with using policy on the receiver, but that's more work and sophistication in bgperf I don't have. 

For now, the best bet is to use BIRD as generator and not use Exa.


So does BIRD have a problem with many neighbors? **Yes**, and I didn't see that until BIRD was also the tester.

## BIRD as tester / generator with many many neighbors
This is a test that is harder than I did previously with ExaBGP. I couldn't get Exa to do 1000 prefixes for this many neighbors. This might not be a realistic set of tests. I don't know if people have 1000 prefixes from 1500 neighbors. But as we'll see, it shows interesting results.

![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_90hold_elapsed.png)

BIRD is considerably worse than everything else; I had forgotten about previous results. Looking back at [the second post](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/) in the section about extreme tests, BIRD does much worse than everybody else at 1000 neighbors, 10 prefixes, or 500+ neighbors, 1000 prefixes. That's what we are seeing here as well. I had just forgotten those results. It's just that with BIRD as a tester it's faster and easier to test these extremes.

Though it's hard to see in this graph because of BIRD's outsized times, but RustyBGP looks like it got better than it was before. It's still slower than FRR, but not 10x slower.


OpenBGPD fails at 1000n_1000p because it runs out of memory on my 64 GB machine and is killed off by the Linux kernel.

![memory usage](/assets/images/2021-09-bgp-episode4/bgperf_90hold_max_mem.png)

BIRD is using a lot more memory than we've see it. Actually, I think something weird is going on in the way bgperf is measuring memory in this case that I haven't seen before. But I only see this for BIRD. 

![free memory](/assets/images/2021-09-bgp-episode4/bgperf_90hold_min_free.png)

This graphs shows the minimum free on the 64 GB machine I'm using for this test. with BIRD there's lots of free memory. Whereas with OpenBGPD and RustyBGP the min free lines up with the amount free. I'm not sure what's going on here, it's weird.

![tester errors](/assets/images/2021-09-bgp-episode4/bgperf_90hold_tester_errors.png)

These errors are counted based on grepping the BIRD tester logs. Actually, more tests have errors, but all these graphs only show tests that finish. So any test that FAILED might also have tester errors.
```
grep RMT /tmp/bgperf/tester/*.log | grep -v NEXT_HOP
```
Usually these errors are hold timers expiring

### BIRD Tester data

<script src="https://gist.github.com/jopietsch/d3e6ad7d946229d2bb967613012529b6.js"></script>
## ExaBGP many neighbors

Just to make sure that the other changes made don't give us different results, also ran similar tests using ExaBGP as the tester.

![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_lots_exa_elapsed.png)

ok, again, Exa as tester is really masking what's going on.

Except, I cannot get RustyBGP to work with ExaBGP as the tester. I just get
```
$ more /tmp/bgperf/rustybgp/rustybgp.log
Hello, RustyBGPd (32 cpus)!
10.10.0.2: down std::io::Error
10.10.0.3: notification 5 2
```
I think that means a BGP Error (Notification) code of 5,2, which would be FSM, Receive Unexpected Message in OpenConfirm State.

I did no debugging. Exa works with all the other BGP stacks just fine, and RustyBGP works with the other testers, so I don't know what's going on.

# MRT Tests

These tests replay real internet routing tables. The specific MRT file used for these tests was [http://archive.routeviews.org/bgpdata/2021.08/RIBS/rib.20210801.0000.bz2](http://archive.routeviews.org/bgpdata/2021.08/RIBS/rib.20210801.0000.bz2)
## Bgpdump2 again

To see if RustyBGP can do better than it did before. These results might not be exactly the same as before because bgperf now checks to see if things are stalled or prefixes count is dropping.

![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_bgpdump_elapsed.png)

Yep, RustyBGP is considerably better than before and now the fastest in this test. 

Why do OpenBPGD and BIRD have less results than before? It's because they had test failures. The OpenBGPD failures are the issues I noted last time.

![memory usage](/assets/images/2021-09-bgp-episode4/bgperf_bgpdump_max_mem.png)

![free memory](/assets/images/2021-09-bgp-episode4/bgperf_bgpdump_min_free.png)

![max cpu](/assets/images/2021-09-bgp-episode4/bgperf_bgpdump_max_cpu.png)

Finally we are seeing what I'd hoped to see when I started this, a BGP stack that used more CPU resources and was faster than the others. **woot woot**.


![tester errors](/assets/images/2021-09-bgp-episode4/bgperf_bgpdump_tester_error.png)

Why aren't there any tester errors? That's because any test that FAILED, we don't put in graphs. But if we look at the data directly for all the FAILED, we can see that for all the OpenBPGD fails, it did notice errors (which is the last column before the FAIL.)

```
$ grep FAIL results/bgpdump.csv
openbgp,openbgp,OpenBGPD 7.1,30,800000,800000,877885,0,270,3,267,292.84,192,11.266,74,41.129,,2021-09-11,32,62.82GB,1,FAILED,FAILED: stuck received count 877885 neighbors_checked 29
openbgp,openbgp,OpenBGPD 7.1,40,800000,800000,878406,0,412,3,409,441.48,164,14.638,74,35.855,,2021-09-11,32,62.82GB,2,FAILED,FAILED: stuck received count 878406 neighbors_checked 39
bird,bird,v2.0.8-59-gf761be6b,50,800000,792000,871147,7,190,3,187,233.67,102,4.162,76,44.243,-s,2021-09-11,32,62.82GB,0,FAILED,FAILED: dropping received count 871147 neighbors_checked 11
openbgp,openbgp,OpenBGPD 7.1,50,800000,800000,878406,0,570,3,567,606.06,171,18.505,77,29.774,,2021-09-11,32,62.82GB,2,FAILED,FAILED: stuck received count 878406 neighbors_checked 49
```

Why did these fail and is it important? 

I don't know what's going on with BIRD here. Usually a drop in received counts is because neighbors connections are closed. I can't find any evidence of that here with BIRD. It doesn't drop by much, but it does drop for at least 10 seconds.


### OpenBGPD vs bgpdump2

Last time I noticed that with Bgpdump2 as the tester, OpenBGPD as the target and 30 neighbors, OpenBGPD stalled and then dropped a neighbor. I still don't know why, but I've added a new errors column in the stats. For BIRD and Bgpdump2 as testers, it will grep the tester logs to see if there are errors and count them. However, there is something going on. It appears bgpdump2 is sending all the data twice. It sends it, waits several seconds and sends it again. This seems to be happening with any and every target. This second time it sends it to openbgpd, for some reason on one of the testers is fails.

The client has this in the log:
```
Sep 17 23:01:26.351754 Read notification message (3), length 21 from 10.10.255.254
Sep 17 23:01:26.351764 Notification Error: UPDATE Message Error (3), Malformed AS_PATH (11)
```
I did finally get logging working for OpenBGPD and so it's logs have:
```
neighbor 10.10.0.32: bad path, starting with 64550, enforce neighbor-as enabled
neighbor 10.10.0.32: sending notification: error in UPDATE message, AS-Path unacceptable
neighbor 10.10.0.32: state change Established -> Idle, reason: Fatal error
neighbor 10.10.0.32: state change Idle -> Active, reason: Start
neighbor 10.10.0.32: state change Active -> OpenSent, reason: Connection opened
neighbor 10.10.0.32: Received multi protocol capability:  unknown AFI 1, safi 4 pair
neighbor 10.10.0.32: Received multi protocol capability:  unknown AFI 2, safi 4 pair
neighbor 10.10.0.32: state change OpenSent -> OpenConfirm, reason: OPEN message received
neighbor 10.10.0.32: state change OpenConfirm -> Established, reason: KEEPALIVE message received
```

I don't know what in that MRT through bgpdump2 makes OpenBGPD find this, where no other stack does and using GoBGP to playback also doesn't have the problem. Continued mystery.


### bgpdump2 data

<script src="https://gist.github.com/jopietsch/c4bff9381f5c93b68c20b3f794f43072.js"></script>

## GoBGP MRT
![elapsed time](/assets/images/2021-09-bgp-episode4/bgperf_gobgp-mrt_elapsed.png)

![memory usage](/assets/images/2021-09-bgp-episode4/bgperf_gobgp-mrt_max_mem.png)

![free memory](/assets/images/2021-09-bgp-episode4/bgperf_gobgp-mrt_min_free.png)

![max cpu](/assets/images/2021-09-bgp-episode4/bgperf_gobgp-mrt_max_cpu.png)


The failure of RustyBGP at 30 neighbors is because my 64 GB machine ran out of memory.

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

What should BGP Hold Timer be for a test like this? The default is 90 seconds, but if the test doesn't even take 90 seconds then we aren't testing if hold timers will work. Should we expect aggressive hold timers to be able to work? I don't know. I started some tests here, but am not sure what they mean yet. Probably the beginning of another investigation.

I've not been paying attention to Hold Timers before this. 

While not exactly the same data, let's compare how many tests failed with default timers vs 5 second hold timer.

he columns that we are showing here are free memory in the machine, tester errors and if the test failed. What we are looking or is if memory gets really low. Other than the above time OpenBGPD runs out of memory, running out of memory and being killed off is not the problem we see here.

Default Timers:
```
$ awk 'BEGIN{FS=","};{print $1 "," $4 "," $5 "," $16 "," $21 "," $23}' lots_bird.csv|grep FAIL
bird -s,500,1000,59.699,0,FAILED: stuck received count 495000 neighbors_checked 495
bird -s,1000,100,60.039,2,FAILED: stuck received count 75400 neighbors_checked 754
bird -s,1000,1000,58.138,11,FAILED: stuck received count 961000 neighbors_checked 961
openbgp,1000,1000,0.012,1000,FAILED: dropping received count 0 neighbors_checked 1000
bird -s,1500,100,58.64,0,FAILED: stuck received count 100000 neighbors_checked 1000
```

The BIRD failures are the ones are the results that we talked about at the beginning of this post and the OpeBGPD is because it ran out of memory. 



5 second timers
```
$ awk 'BEGIN{FS=","};{print $1 "," $4 "," $5 "," $16 "," $21 "," $23}' birt_many_5.csv|grep FAIL
bird -s,500,100,59.411,0,FAILED: stuck received count 47300 neighbors_checked 473
rustybgp,500,1000,50.673,279,FAILED: dropping received count 221000 neighbors_checked 221
bird -s,1000,100,58.536,13,FAILED: stuck received count 97700 neighbors_checked 977
rustybgp,1000,100,55.833,324,FAILED: dropping received count 67600 neighbors_checked 676
bird -s,1000,1000,56.837,13,FAILED: stuck received count 1000000 neighbors_checked 0
frr,1000,1000,56.073,0,FAILED: stuck received count 1000000 neighbors_checked 728
frr 8,1000,1000,56.035,0,FAILED: stuck received count 1000000 neighbors_checked 0
openbgp,1000,1000,0.048,1000,FAILED: dropping received count 0 neighbors_checked 0
rustybgp,1000,1000,2.922,1553,FAILED: dropping received count 631588 neighbors_checked 222
bird -s,1500,10,59.528,1,FAILED: stuck received count 14020 neighbors_checked 1101
rustybgp,1500,10,58.788,14,FAILED: dropping received count 14860 neighbors_checked 1486
bird -s,1500,100,59.177,67,FAILED: stuck received count 150000 neighbors_checked 653
frr 8,1500,100,58.367,0,FAILED: stuck received count 150000 neighbors_checked 0
openbgp,1500,100,28.934,0,FAILED: stuck received count 150000 neighbors_checked 0
rustybgp,1500,100,48.041,897,FAILED: dropping received count 138811 neighbors_checked 720
```
Many Many more failures here. I don't have a specific pattern here. You can see that every stack had some problems. I think RustyBGP has the most problems with the hold timer set to 5 seconds. BIRD doesn't really have more failures than before though it does have more hold timer experations than before.


# What did we learn
I thought I'd learned about a BIRD performance issue, but it turns out I already knew that, but using BIRD to measure BIRD makes it more obvious. Also, are the tests that BIRD did bad on realistic? Unlikely. It's finding an edge case in which BIRD performs badly, but you are unlikely to get to that place.

BIRD as a tester is a tester that takes less resources, so I've made it the default tester now. it's more pleasant. But Exa had a problem with RustyBGP, so it might be good to use it as one more compatibility test.

I'm not yet sure what we learned about hold timers. Ideally 5 second hold timers for containers that are on the same computer shouldn't be a problem; maybe I'm being naive? 

bgperf is a bit better every time.
## RustyBGP

Did RustyBGP get better? Yes, much better. It's the fastest in the MRT playback tests. It's not as fast as FRR in the many neighbors tests, but those are less realistic. We finally have a BGP stack that can take advantage of modern hardware. ( I mean, that I'm measuring. I know there are others that probably do also, but they are commercial and I just haven't gotten there yet.) However, it also has some weird problem with Exa as the tester that I have not tracked down. 

I think it has a problem with hold timers. At least I can get a lot of hold timers to expire and I think that's bad, but am not sure.


## BIRD

BIRD is much slower than expected when there are many neighbors (> 500) neighbors. At least it gets much worse than expected.
## OpenBGPD

As noticed before it has some reset on bgpdump2 that I have tracked down.

It also uses more memory, and a lot more memory per neighbor. It's been noticed in previous comments that is because it has a separate ADJ-RIB-IN for every neighbor rather than a shared one with pointers. I think the implication is that it might be for safety. Whatever the reason, if you have lots of neighbors it will cost you. Possibly it doesn't matter, but it might.

## Which stack to use

I still don't know. If any of my previous conclusions made you think you should pick one over the other, I'm sorry. I'm just trying to measure and find differences. This post especially is just targeted to specific tests. If you have more than 500 neighbors with BIRD and they each have lots of routes then maybe you should be worried.

FRRouting is a strong performer in all of these tests. It's interesting that for IXP style work BIRD and OpenBGPD seem to be the only open source option. I don't know if that is because several years ago Quagga/FRRouting was poor in performance here, that at least seems to have been the story. 

Of course, there are still big pieces of BGP stack performance, especially routing policy and filtering, that I've not gotten to and are extremely important. 

I wouldn't use GoBGP if performance is an issue. RustyBGP is still immature but promising. 
