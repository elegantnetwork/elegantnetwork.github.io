---
layout: post
comments: true
author: Justin Pietsch
title: Suzieq has a GUI
excerpt: Our 0.8 release has (finally!) added a GUI.
description: Suzieq is for Networking Understanding. Now with a GUI!
---
We haven't blogged for almost three months. We've just been busy working on [(Suzieq)](https://github.com/netenglabs/suzieq). We've been adding features and figuring out where we want it to go. We haven't blogged about Suzieq for even longer, mostly because we're not sure that's what people are interested in compared to other content. But it's time to get back to Suzieq. Our 0.8 release has (finally!) added a GUI.

We've had a hard time figuring out how to best describe what Suzieq is and why it's useful. We started with Network Observability, but after our last blog post on [Network Observability](https://elegantnetwork.github.io/posts/observability/), it doesn't look like that term resonates with Network Engineers and Operators.

Suzieq allows you to have better understanding of your network. It allows you to do deep dives into the data in your network


# Why should you care about Suzieq
Networks are complicated and complex, they are hard to understand well, and that lack of understanding leads to lots of problems. It makes it much harder to troubleshoot, it's hard to find things, and it leads to less reliable networks. Suzieq gathers the most important data from your network that you use when log into devices to understand and debug problems, and puts them all in one place.

Suzieq allows analysis through a CLI, python objects, REST API, and now finally a GUI. The GUI does make it easier to present related data together, and of course, to show pictures, which are useful from time-to-time. We started with a CLI because we are not GUI/UI people, and we wanted to build something useful as soon as possible. Over time we've also added a [REST API(https://suzieq.readthedocs.io/en/latest/rest-server/)] so that other systems can get access the Suzieq Analysis. We've now gotten to a GUI to demonstrate Suzieq's usefulness.

Suzieq makes it really easy to explore your network, add asserts about correctness, dive into the details. 

# Examples

## Status
With the GUI, Suzieq how has a way to show the status of the network. We expect this to get better, but we can at least show potential. ![Suzieq Status](/assets/images/2020-12-suzieq-gui/status-2.png)

## Xplore
Suzieq has always been about exploring your network. Some of the pieces of that are the regular table output, but it's also had the ability to summarize data and to provide histograms of the data. With the GUI Suzieq can show all of that together and it does a nice of representing how these different presentations are useful together.

Here you can see a view of the devices. There is a summary table at the top left to help you make sense of all the data. ![devices](/assets/images/2020-12-suzieq-gui/devices-gui.png) The Distribution Count section lets you pick any of the columns that you want to look at histograms.


In this example, we look at routes. In the summary table you can see how many IPv4 and V6 routes, how many are host routes, a distribution of routes per device, etc. ![routes](/assets/images/2020-12-suzieq-gui/routes-xplore.png)

One fun example is this data with a router that has a full IPv4 internet table. You can see the prefix length distribution. ![internet routes](/assets/images/2020-12-suzieq-gui/routes-internet.png)

There are so many things to Xplore with Suzieq. 

## Path Trace
Suzieq has always had the ability to show all the paths between two source and dest IPs. Dinesh has continued to improve it's ability to deal with all the different aspects of forwarding such as overlays. With the GUI, we can even show you what it looks like and you can spot inconsistencies in your path. ![path trace](/assets/images/2020-12-suzieq-gui/path-gui.png)

## Search

One of the most common uses for suzieq is to find things in the network, like MAC or IP address. While you can use the regular Xplore and pick the right table, we have a convenient search to look through any of those tables. It defaults to address table, but you can have it also search route and arpnd. ![search](/assets/images/2020-12-suzieq-gui/search-route.png)

## Asserts
Another really powerful tool that Suzieq has are asserts. Asserts things that should be true, and then it can show violations. On this version of the GUI we've included the assert output on the same Xplore page for those services that have asserts. ![interface assert](/assets/images/2020-12-suzieq-gui/interfaces-with-assert.png)

# Where we are going

We keep working on Suzieq. We have been working to shore up it's foundation and also take advantage of what it has. Two areas that we haven't really taken advantage of yet are time asserts. Suzieq stores everything with a timestamp and can investigate things overtime, rather than just a snapshot at a time. This enable really powerful things like what just changes in the last 2 hours? While suzieq has all the data, right now it's not easy to really understand what it's telling you.

The other, as I mentioned are asserts. With the data that Suzieq has, it can do some sophisticated analysis to understand how your network is working. We have a smattering of asserts, some are trivial (but really useful), like MTU, and some are complex, like the EvpnVni asserts. One idea is to have an asymetric path assert; if the two different directions have different number of hops that probably isn't what you expected.