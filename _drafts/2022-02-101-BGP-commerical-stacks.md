---
layout: post
comments: true
author: Justin Pietsch
title: Performance testing of Commercial BGP
excerpt: "We've gotten to the place that it would be nice to see how some commercial stacks compare."
description: We've been focused on Open source, it's time to try out Commercial BGP
---

- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Follow-up Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)
- 4th Post [Bird on Bird, Episode 4 of BGP Perf testing ](https://elegantnetwork.github.io/posts/bird-on-bird-bgp-perf-episode4)
- 5th Post [BGP Performance 5 -- 1000 full internet neighbors](https://elegantnetwork.github.io/posts/bgp-perf5-1000-internet-neighbors/)
- 6th Post [BGP Performance testing with filtering](https://elegantnetwork.github.io/posts/bgperf-first-try-at-filtering/)

[I did these experiments in the beginning of December 2021, it's taken some months to get to actually publishing this post.]

So far in these posts it's been all open source because I wanted to understand the state of open source stacks, and because it's technically and legally easier. We've gotten to the place that it would be nice to see how some commercial stacks compare. I stared with the stacks that are containers: Juniper Junos cRDP and Arista EOS cEOS. I have finally gotten permission from Juniper to publish results; Arista, like Juniper, forbids publishing without permission and I have not gotten permission (I haven't tried because I don't have a direct contact with Juniper.)

![Elapsed time](/assets/images/2021-12-bgp-7/bgperf_benchmark-mrt-publish_elapsed.png)

This was tested on my AMD 3950 32 core, 64GB RAM machine. Junos is uni-threaded Juniper cRPD and rs-16 sets the thread count to 16 on multi-threaded cRPD. 

Junos with uni-threading is better than FRR or BIRD. That's interesting and good to know.

As mentioned above so that we can see the results between uni-threading and multi-threading. Play around the number of threads: adding more threads doesn't mean that you will get better performance. I didn't see any difference in performance with 16 or 31 threads on my AMD. I didn't do less threads which would be  interesting.

Juniper isn't faster than RustyBGP when using multi-threading. However, it does use less resources which most likely means that with less CPU it would be faster, but I didn't do that test. 

![Max CPU](/assets/images/2021-12-bgp-7/bgperf_benchmark-mrt-publish_max_cpu.png)

## Testing Commercial BGP Software
I started this project because I wanted to see how well BGP open source stacks performed, how well stacks in modern languages worked, and if I could come up with a test system that would be useful. After publishing the first post, I realized I had done an unsatisfactory job, and the rest of the posts have been working on better methods of testing and reporting. I've added some open source stacks, but the goal has been to do a good job before really expanding the pool of tested BGP software. Now that there is some filtering, it's time to move forward.


Just to back up, how important is this BGP perf testing? For the most part, BGP performance is good enough, but there are use cases in which you need much faster BGP. In other words, it's usually not important, but when it is important, it's important. :)

It's much easier to test containers. Well, [bgperf2](https://github.com/netenglabs/bgperf2) was originally created to build BGP software, produce the containers, wire up the networking, and do performance testing. Adding pre-packed containers is a little more difficult than just a BGP daemon, but isn't too hard. 

I don't have a company I am doing this for, so I had to create new accounts to get the software. Juniper did not allow me to get the software with no commercial relationship, however I do know somebody at Juniper who provided me software and a license. For Arista I created an account, and in the process of downloading the software I read the license agreement and ran into what I expected: they do not allow me to publish any performance data. 

I can't publish all the results: I actually thought I wouldn't be able to publish any, but after waiting two months I got the okay from Juniper. I understand why, but it's deeply frustrating. How can we make good engineering decisions if we can't get good data to understand our decisions. **I can't publish all the results, but I did make bgperf2 so that you can do your own testing.** All the changes to bgperf2 to test these two stacks is published and works. I do not yet have filter support.

I did find interesting things that I did not know. Both BGP stacks do have multi-threading/processing/core ability, but not by default. And I believe that both of them are using a different BGP software stack for uni-thread vs multi-thread, which is why you have to set things explicitly to be multi-threaded: there are different features and tradeoffs when using multi-threading BGP.

I focused on bgpdump2 playback of [Routeviews](http://www.routeviews.org/routeviews/) MRT files just like the other "internet" style tests I've done. As mentioned, I don't have filter support so haven't done any filter performance testing.

For multi-thread performance, in my tests multi-threading makes a big difference. It's worth it if performance is a problem. 

One interesting thing is that Juniper requires you to set the number of threads to use, and Arista has a hardcoded number that doesn't seem to change no matter the hardware you are on. I don't know why. 

[Instructions on testing commercial BGP stacks](https://github.com/netenglabs/bgperf2/blob/master/README.md#targets)



## Arista

I would like to publish Arista data, but cannot. If you have Arista and care about BGP performance, bgperf2 should be all setup for you to do your own tests. I would encourage you to do so.
## Data
<script src="https://gist.github.com/jopietsch/439f80528f03e50d246d83cd4184f7ec.js"></script>
## bgperf2

As an aside, I've moved bgperf to [bgperf2](https://github.com/netenglabs/bgperf2) so that it it it's own project and not just a github fork of bgperf. This should make it possible to file issues and submit PRs.

## P.S.
I've moved the bgperf that I used to a real project: [bgperf2](https://github.com/netenglabs/bgperf2). This means that it can easily take PRs and file Issues. 
## P.P.S.
I'm about to start a new job, I have no idea what they'll think of these blog posts. These results and opinions have nothing to do with any company that I have worked at or will in the future. 

It's very likely that my energy for bgperf2 and bgp stack perf testing will be taken up by my new job.