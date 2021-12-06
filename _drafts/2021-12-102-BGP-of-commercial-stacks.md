---
layout: post
comments: true
author: Justin Pietsch
title: Performance testing of Commercial BGP
excerpt: "(I) ran into what I expected: they do not allow me to publish any performance data. "
description: We've been focused on Open source, it's time to try out Commercial BGP
---

- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Follow-up Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)
- 4th Post [Bird on Bird, Episode 4 of BGP Perf testing ](https://elegantnetwork.github.io/posts/bird-on-bird-bgp-perf-episode4)
- 5th Post [BGP Performance 5 -- 1000 full internet neighbors](https://elegantnetwork.github.io/posts/bgp-perf5-1000-internet-neighbors/)
- 6th Post [BGP Performance testing with filtering](https://elegantnetwork.github.io/posts/bgperf-first-try-at-filtering/)

I started this project because I wanted to see how well BGP open source stacks performed, how well stacks in modern languages worked, and if I could come up with a test system that would be useful. After publishing the first post, I realized I had done an unsatisfactory job, and the rest of the posts have been working on better methods of testing and reporting. I've added some open source stacks, but the goal has been to do a good job before really expanding the pool of tested BGP software. Now that there is some filtering, it's time to move forward.

Just to back up, how important is this BGP perf testing? For the most part, BGP performance is good enough, but there are use cases in which you need much faster BGP. In other words, it's usually not important, but when it is important, it's important. :)

It's much easier to test containers. Well, [bgperf](https://github.com/jopietsch/bgperf) was originally created to date BGP software, produce the containers, wire up the networking, and do performance testing. Adding pre-packed containers is a little more difficult than just a BGP daemon, but isn't too hard. 

![no results elapsed time](/assets/images/2021-12-bgp-7/commercial-perf.jpg)

I stared with the stacks that are containers: Juniper Junos cRDP and Arista EOS cEOS. I don't have a company I am doing this for, so I had to create new accounts to get the software. Juniper did not allow me to get the software with no commercial relationship, however I do know somebody at Juniper who gave me software and a license. For Arista I created an account, and in the process of downloading the software I read the license agreement and ran into what I expected: they do not allow me to publish any performance data. 

I understand why, but it's deeply frustrating. How can we make good engineering decisions if we can't get good data to understand our decisions. **I can't publish results, but I did make bgperf so that you can do your own testing.** All the changes to bgperf to test these two stacks is published and works. I do not yet have filter support, but expect to in the near future.

I did find interesting things that I did not know. Both BGP stacks do have multi-threading/processing/core ability, but not by default. And I believe that both of them are using a different BGP software stack for uni-thread vs multi-thread, which is why you have to set things up.

I did the tests I'd like to be able to show you, but I can't. I focused on bgpdump2 playback of [Routeviews](http://www.routeviews.org/routeviews/) MRT files just like the other "internet" style tests I've done. As mentioned, I don't have filter support so haven't done any filter performance testing.

For uni-thread performance, neither is appreciable faster than BIRD or FRR, and one is noticeably slower.

For multi-thread performance, in my tests multi-threading makes a big difference. It's worth it if performance is a problem. 

One interesting thing is that Juniper requires you to set the number of threads to use, and Arista has a hardcoded number that doesn't seem to change no matter the hardware you are on. I don't know why. 

[Instructions on testing commercial BGP stacks](https://github.com/jopietsch/bgperf/blob/master/README.md#targets)

## Juniper

Play around the number of threads: adding more threads doesn't mean that you will get better performance.

## Arista