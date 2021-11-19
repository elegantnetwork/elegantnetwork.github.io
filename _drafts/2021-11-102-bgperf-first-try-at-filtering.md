---
layout: post
comments: true
author: Justin Pietsch
title: BGPerf testing filtering
excerpt:
description: 
---

- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Follow-up Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)
- 4th Post [Bird on Bird, Episode 4 of BGP Perf testing ](https://elegantnetwork.github.io/posts/bird-on-bird-bgp-perf-episode4)
- 5th Post [BGP Performance 5 -- 1000 full internet neighbors](https://elegantnetwork.github.io/posts/bgp-perf5-1000-internet-neighbors/)

The number one request I get for BGP Performance testing is to do filtering testing. There are several difficulties with this, and I'll discuss them below, but that is why I've put it off so long. I have some results and I'd like to see what the audience thinks about the testing methodology and if it provides the visibility that we are all hoping for.

I need help. Is this the right way to represent filtering for these platforms?

Obviously one of the problems is trying to create filtering that can be used across all platforms and will be representative of what people are actually using. I started with https://bgpfilterguide.nlnog.net. I'm currently only using two filters that are mostly the same. The first, called "transit", is the filter that you would use if the peer was transit, and the other is called "ixp" if your peer was in your IXP. The difference between the two is that the ixp filter filters our transit ASNs. 

Just to refresh how to read the key: "20n_800000p_transit" means 20 neighbors, 800000 prefixes per neighbor and it's using the transit filter.

## Internet Routes

We'll start with internet routes using bgpdump2 playing back mrt data from [Routeviews](http://www.routeviews.org/routeviews/). 

![internet routes elapsed](/assets/images/2021-11-bgp-6/bgperf_filter-bgpdump_elapsed.png)

There are several things to notice, some we'll discuss later in this post. First off, for the most part, the "transit" filter takes more time than the non-filtered run. This is because it does go through the filters, but it doesn't filter many prefixes. Since these are from real internet peers, there are not a lot of prefixes that you'd want to filter. The "ixp" filter on the other hand does filter more prefixes out, so there is less total elapsed time because there are less prefixes being considered.

For now, ignore that rustybgp is missing data for the "ixp" filter. I'll address that below.

I've added a new graph ![internet routes monitor](/assets/images/2021-11-bgp-6/bgperf_filter-bgpdump_monitor_prefixes.png) 

This shows the amount of prefixes that end up at the monitor container. We'll discuss this below, but notice that for the same filters, different NOSes have different number of prefixes that get to the monitor.

The other thing to notice is that the "transit" filter doesn't filter very much and the "ixp" filter does, which explains the elapsed time differences.

## Large number of neighbors, small number of prefixes
![bird generator elapsed](/assets/images/2021-11-bgp-6/bgperf_filter-bird-1000_elapsed.png)

![bird routes monitor](/assets/images/2021-11-bgp-6/bgperf_filter-bird-1000_monitor_prefixes.png)


##  The difficulty in testing filtering
As mentioned above, I've been asked for these tests and I've put them off. There have been some difficulties getting it to work. Number one is figuring out what is representative and that will work on all BGP stacks, including where we might go in the future. The big issue for me is that I now have to do a bunch of per BGP stack work to make this fair, and while I've done my best, let me know what I've missed.

As discussed, I started with https://bgpfilterguide.nlnog.net and I've tried to make the filter for each NOS equivalent. If this doesn't match the kind of filtering you want to see let me know.


Of course, you have to match traffic with filters to see if they work.

### not all NOSes filter the same

As far as I can tell, the filters for BIRD, FRRouting and OpenBGPD should be the same, so why are there different amounts of prefixes filtered? I have no idea. Please tell me if I'm dong something wrong.

To look at the filters used:
* [BIRD](https://github.com/jopietsch/bgperf/blob/490452fe947c94f9eb87e4f45bb789514f6e11b1/filters/bird.conf)
* [FRRouting](https://github.com/jopietsch/bgperf/blob/490452fe947c94f9eb87e4f45bb789514f6e11b1/filters/frr.conf)
* [OpenBGPD](https://github.com/jopietsch/bgperf/blob/490452fe947c94f9eb87e4f45bb789514f6e11b1/filters/openbgp.conf)
* [RustyBGPD](https://github.com/jopietsch/bgperf/blob/490452fe947c94f9eb87e4f45bb789514f6e11b1/filters/rustybgpd.conf)

You might notice that I've commented out filtering of 0.0.0.0/8 and > /24 filtering. That is so that I could do the BIRD-generator tests with small number of prefixes. It generates prefixes in that range, so they'd all get removed which isn't interesting.

For the tests with the BIRD-generator, while the "ixp" filter still filtered out prefixes with transit ASNs, there are none in that data, so I've included /24 filtering for "ixp".



### how to count all received prefixes
Ok, this is one of the trickiest pieces here, but we have to get to how BGPerf works and how it decides that a benchmark run is done. There are three main groups of containers: target, tester (which with bgpdump is more than one container), and monitor. The monitor sends no prefixes, it is used to measure prefixes being sent. 

When I fork BGPerf, it decided a test done after the monitor had received the expected number of prefixes. Because different BGP stacks receive and send routes in different order, I changed it so that on the target it needed to have received the expected number of prefixes from each tester and still that it has received all the prefixes at the monitor.

Maybe you see the problem with filter testing: when filtering, we now don't know when all the prefixes that will be sent have been received because we don't know what will get filtered. So I needed to add a new counter that measures the amount of prefixes received at the target. However, this is tricky. While all BGP stacks will tell you how many prefixes it has received after filtering, few (I've only found BIRD so far), will also tell you how many were received before being filtered or rejected.

This is super helpful output that I wish all stacks would show: it comes from birdc 'show protocols all' and is showing just one neighbor.

```
    Route change stats:     received   rejected   filtered    ignored   accepted
      Import updates:        1599980          0    1493658      53161      53161
      Import withdraws:           20          0        ---         20          0
      Export updates:          53161      53161          0        ---          0
      Export withdraws:            0        ---        ---        ---          0
```
So what do I do about the stacks that don't have this information? I got a helpful tip from Donald Sharp of FRRouting fame. FRRouting will show End-of-RIB in the log if you turn on debugging, so that's what I did for FRRouting. For OpenBGPD, it has an eor counter in it's output that BGPerf looks for. For rustybgp it has a counter called accepted and received, but they show the same output, so I haven't figured out how to measure that. That's why rustybgp doesn't show data with IXP filters in the graphs above; BGPerf has no way to know when the test has been finished successfully.

Figuring out how to see that all the intended prefixes have been received at the target is a pain. 

## Conclusion
lots of questions
* Is this what you hoped to see?
* what am I missing?
If you have opinions about this, I'd rather see specific examples of filters than vague descriptions like what and IXP would use. 



