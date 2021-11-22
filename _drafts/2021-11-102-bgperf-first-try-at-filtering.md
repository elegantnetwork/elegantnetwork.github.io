---
layout: post
comments: true
author: Justin Pietsch
title: BGPerf testing of filtering
excerpt: The number one request I get for BGP Performance testing is to do filtering tests.
description: First results of testing of filter with BGP stacks to get feedback.
---

- 1st Post [Comparing Open Source BGP Stacks](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-stacks/)
- 2nd Post [Follow-up Measuring BGP Stacks Performance](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)
- 3rd Post [Comparing Open Source BGP stacks with internet routes](https://elegantnetwork.github.io/posts/comparing-open-source-bgp-internet-routes)
- 4th Post [Bird on Bird, Episode 4 of BGP Perf testing ](https://elegantnetwork.github.io/posts/bird-on-bird-bgp-perf-episode4)
- 5th Post [BGP Performance 5 -- 1000 full internet neighbors](https://elegantnetwork.github.io/posts/bgp-perf5-1000-internet-neighbors/)

The number one request I get for BGP Performance testing is to do filtering testing. There are several difficulties with this, and I'll discuss them below, but that is why I've put it off so long. I have some results and I'd like to see what the audience thinks about the testing methodology and if it provides the visibility that we are all hoping for.

I'm not sure that what I've created tests filtering in the way that people want to see, so I'm producing results now in the hope of getting some feedback to make things better.

Obviously one of the problems is trying to create filtering that can be used across all platforms and will be representative of what people are actually using. I started with https://bgpfilterguide.nlnog.net; below I have pointers to the specific filters I'm using. I'm currently only using two filters that are mostly the same. The first, called "transit", is the filter that you would use if the peer was transit, and the other is called "ixp" if your peer was in your IXP. The difference between the two is that the ixp filter filters our transit ASNs. 

Just to refresh how to read the key: "20n_800000p_transit" means 20 neighbors, 800000 prefixes per neighbor and it's using the transit filter.

# Internet Prefixes

We'll start with internet routes using bgpdump2 playing back mrt data from [Routeviews](http://www.routeviews.org/routeviews/). 

![internet routes elapsed](/assets/images/2021-11-bgp-6/bgperf_filter-bgpdump_elapsed.png)

There are several things to notice, some we'll discuss later in this post. First off, for the most part, the "transit" filter takes more time than the non-filtered run. This is because it does go through the filters, but it doesn't filter many prefixes. Since these are from real internet peers, there are not a lot of prefixes that you'd want to filter. The "ixp" filter on the other hand does filter more prefixes out, so there is less total elapsed time because there are less prefixes being considered.

For now, ignore that rustybgp is missing data for the "ixp" filter. I'll address that below.

I've added a new graph ![internet routes monitor](/assets/images/2021-11-bgp-6/bgperf_filter-bgpdump_monitor_prefixes.png) 

This shows the amount of prefixes that end up at the monitor container. We'll discuss this below, but notice that for the same filters, different BGP stacks have slight differences in the number of prefixes that get to the monitor.

The other thing to notice is that the "transit" filter doesn't filter very much and the "ixp" filter does, which explains the elapsed time differences.

# Large number of neighbors, small number of prefixes

Now I want to show results from small number of prefixes, and many neighbors. 

![bird generator elapsed](/assets/images/2021-11-bgp-6/bgperf_filter-bird-1000_elapsed.png)

The first thing is that it's weird that Both FRRs have higher elapsed time with 0 prefixes getting through. Do not know what that means. For BIRD and OpenBGPD, when it gets to 0 prefixes they are really fast.

![bird routes monitor](/assets/images/2021-11-bgp-6/bgperf_filter-bird-1000_monitor_prefixes.png)

Nothing interesting here; just confirming what was expected: "transit" filter filers nothing and "ixp" filter lets nothing through.

#  The difficulty in testing filtering
As mentioned above, I've been asked for these tests and I've put them off. There have been some difficulties getting it to work. Number one is figuring out what is representative and that will work on all BGP stacks, including where we might go in the future. The big issue for me is that I now have to do a bunch of per BGP stack work to make this fair, and while I've done my best, let me know what I've missed.

As discussed, I started with https://bgpfilterguide.nlnog.net and I've tried to make the filter for each BGP stacks equivalent. If this doesn't match the kind of filtering you want to see let me know.


Of course, you have to match traffic with filters to see if they work.

### What filters am I using


To look at the filters used:
* [BIRD](https://github.com/jopietsch/bgperf/blob/490452fe947c94f9eb87e4f45bb789514f6e11b1/filters/bird.conf)
* [FRRouting](https://github.com/jopietsch/bgperf/blob/49dcf42868dc88cb65f95924c28d4b25cf8ba5b0/filters/frr.conf)
* [OpenBGPD](https://github.com/jopietsch/bgperf/blob/490452fe947c94f9eb87e4f45bb789514f6e11b1/filters/openbgp.conf)
* [RustyBGPD](https://github.com/jopietsch/bgperf/blob/6af19e343633c76e9c00f825d69fa91920d3a9b9/filters/rustybgpd.conf)

You might notice that I've commented out filtering of 0.0.0.0/8 and > /24 filtering. That is so that I could do the BIRD-generator tests with small number of prefixes. It generates prefixes in that range, so they'd all get removed which isn't interesting.

For the tests with the BIRD-generator, while the "ixp" filter still filtered out prefixes with transit ASNs, there are none in that data, so I've included /24 filtering for "ixp".



### How to count all received prefixes
This is one of the trickiest pieces here, but I need to explain how BGPerf works and how it decides that a benchmark test is done. There are three main groups of containers: target, tester (when using bgpdump it is more than one container), and monitor. The monitor sends no prefixes, it is used to measure prefixes being sent. 

When I forked BGPerf, it decided a test was done after the monitor had received the expected number of prefixes. Because different BGP stacks receive and send routes in different order, I changed it so that on the target it needed to have received the expected number of prefixes from each tester and still that it has received all the prefixes at the monitor.

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

# Conclusion
What did we learn? Not sure. I guess that it's possible to do some amount of testing, but it is tricky to know when a test has finished. 

It's interesting that while the data with "transit" filtering does show more elapsed time, it's not a lot more. Because of this, I'm not sure that what I have so far for filtering is what people will see. From this data I don't know that we can really say that one BGP stack is better at filtering than the other. 

FRRouting 8 is faster than 7.5.1.

There are at least more questions that answers here
* Is this what you hoped to see?
* Are these filters representative enough?
* Are the prefixes going to the data representative enough?
* what am I missing?

If you have opinions about this, I'd rather see specific examples of filters than vague descriptions like what an IXP would use. 

Let me know how to make this better. Or send code? :)


