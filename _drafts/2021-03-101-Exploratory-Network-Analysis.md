---
layout: post
comments: true
author: Justin Pietsch
title: Exploratory Network Analysis (XNA)
excerpt: Just like in data analysis, you need to be able to explore your network and find things easily.
description: You need to be able to explore and Find things easily in your network.
---
![Suzieq interface type](/assets/images/2021-03-xna/interface-show-type.png)

In data science there is this idea called Exploratory Data Analysis ([EDA](https://www.itl.nist.gov/div898/handbook/eda/section1/eda11.htm)). When I have a big bunch of data, what’s in it? How do I make sense of it? Where do I even start? In data analysis, this led to the S language, which led to S-Plus and the current favorite analysis language [R](https://www.r-project.org/about.html). In networking I think we have much of the same problem and we don't have ways to systematically approach it. We have to custom build tools and analysis for each question we want to ask, but usually we either do it all manually, relying on humans to remember and process things, including using whiteboards. This seems wildly insufficient, especially in the 2020s.

EDA was defined by John W. Tukey in 1961. In his book Exploratory Data Analysis in 1977 he felt that there is too much emphasis on statistical testing, and not enough on understanding the data. I think we are in a similar situation in which we are not really looking at our network to understand how it operates. We rely on intuition and the general assumption that if users aren't complaining, then everything is okay. The point is that a small number of statistics isn't enough to understand the data (network.) You need to be able to look at the data, ask questions, find answers, ask more questions, etc. Just like in data analysis, not everything you want to examine is a metric. 

I like the points in this article [Why EDA is Crucial for any Data Science Prject](https://www.aismartz.com/blog/why-eda-is-crucial-for-any-data-science-project/).
- Get a better understanding of data
- Understanding data patterns
- Drawing charts and graphs for better understanding
- To get a better understanding of the problem statement


## XNA (eXploratory Network Analysis)

Networks are complicated, have a lot of data, and are hard to understand. How do you explore your network now to find out what protocols you are using, or if is is operating as expected? You look at configs, maybe you look at traffic graphs, and you probably have to log into devices to run show commands. But the configs are complicated and hard to get your head around. And the traffic graphs don't represent how traffic flows or the protocols that ensure a functioning network. You look at a diagram, if it's up to date, at least you hope it is. 

Just like in data analysis, you need to be able to explore your network and find things easily.

I want to talk about this idea in general, and then talk about what we've implemented in [Suzieq](https://www.stardustsystems.net/suzieq/) for XNA. I'd love a discussion about the ideas in general, and any ideas on how to make Suzieq even better at eXploratory Network Analysis. We are accepting PRs :). Suzieq is currently focused on operational state in network devices. In the future it will collect interface counters and other metrics, but that's not where we've started. The reason is that there really aren't good ways to monitor and investigate operational data in networks. Suzieq does not currently answer all the exploratory questions I'd like to ask of a network, but it's a very good start.

Some uses cases:
- Where is this IP address or MAC address?
- How often (and when) does this MAC move?
- Where did this MAC go? How many MACs do I have? Do I have any outliers that move a lot
- What is my topology, L1, L2, L3, per protocol
	- Do my L3 topologies have the same exact shape to my L1/L2?
- Why does my network work this way?
- What does this policy do this? what does it mean for my overall network?
- If I make this change, what will change?
- How many OSes do I have? How many device types?
- How does a packet traverse through my network including L2, L3, and eVPN hops?
- How have things changed over time?
	- What changed in the last 24 hours

Most networks (AFAIK) have reasonable interface monitoring and graphs of these metrics. However, even with counters, there are not good ways to explore your data. You can usually point to a specific interface to see it's utilization, but can you ask more general questions like:
- How many interfaces are ever over 50% utilized?
- What percentage of interfaces 99th percentile is less then 20% utilization?
- If we change our TCP algorithm, will we have less drops in the network? 
	- How will the distribution of drops change?
- If we change our buffers, will we have less drops in the network?
- When will we run out of capacity and where will that be?
- What is the correlation between utilization and drops and is it the same across the network?


We are using standard data science tools in Suzieq like Python, Pandas, and Parquet. We’ve built network engineering and operations applications on top of that, but you can use a Jupyter notebook and do all the analysis yourself if you so desire. 

## Some Examples

Let's say that I'm new to a network. I want to understand what it is in the network and how it works. ![Suzieq device show](/assets/images/2021-03-xna/device.png)
This gives you an idea of what devices are in your network. From the summary table you can see that it's made up of some Ubuntu hosts and some Cumulus routers. From the histogram on the top right, you can see the versions. So this is a pretty simple network, and all the OSes are consistent, which is nice.

OK. So let's look around. Let's look at routing protocols. Starting with OSPF, using the CLI. ![Suzieq OSPF summarize CLI](/assets/images/2021-03-xna/ospf-summarize.png)
This gives you a good summary of OSPF in this network. There are 8 OSPF speakers, all in the same area 0, all in the same VRF. You can see the important OSPF timers as well. That seems straightforward, ok.

How about BGP.![Suzieq BGP summarize GUI](/assets/images/2021-03-xna/bgp-summarize.png)
OK, there are 10 BGP speakers; interestingly different than the number of OSPF speakers. There is iBGP and eBGP, no Route Reflectors, IPv4, no IPv6, and there is EVPN.

We can look at EVPN to get an an overview also. It's symmetric EVPN, some L2 and L3, and no multicast.
![Suzieq EVPN summarize](/assets/images/2021-03-xna/evpnVni-summarize.png)

Now I'm going to jump into some Suzieq features that are Alpha, and ugly, but can be useful. It'd handy to compare topology from different layers in your network. We want to get an idea of the overall physical topology using LLDP. ![Suzieq topology LLDP](/assets/images/2021-03-xna/ospf-ibgp_lldp.png)

Same thing for BGP. You'll notice that there is a link that exists with LLDP from spine02 to exit02 that does not exist in BGP. This is clearly a bug (or the BGP session is down, but in the this case it's a bug in the configuration.) By exploring this data we can see that the network is not the way that we expect.
![Suzieq topology BGP](/assets/images/2021-03-xna/ospf-ibgp_bgp.png)

### Path Trace
This isn't really XNA, but it does allow you to explore your network. Suzieq can show you the forwarding decisions that routers make from a source, destination pair. ![Suzeq path trace](/assets/images/2021-03-xna/path-show-internet.png) That looks weird. There should be a path from spine01 and spine02 to exit01, just like there is to exit02. If you click on the link spine02 (or spine01) to exit02, you will see how the forwarding decisions are made. ![Suzieq path debug](/assets/images/2021-03-xna/spine02-exit02-path.png) You can see that the route was populated by BGP, but it only has one interface out. For some reason spine02 doesn't have a route to 172.16.253.1 that has a nexthop on exit01.

### Route Table

As an example, Suzieq gathers the routing table from every device and allows you to look up that data. You can also ask it to give you a summary of the table at any given time, such as the size of the table. But what if you want to know how fast your routing table is growing. Suzieq doesn’t currently answer that question, but it has the data so that you could answer it yourself in several different ways



## How is this different than observability
We've written about [Network Observability](https://elegantnetwork.github.io/posts/observability/) and it is very related to this area of XNA, it's just coming from a different field (compute devops vs Statics and data analysis). Observability focuses on being able to ask arbitrary questions in your network just like XNA. I would say that a good observability platform allows you to do this kind of Exploration. The point of EDA and XNA is the exploration that is needed.


## Conclusion
We all need good ways to explore what's in our network. We need better tools to allow us to do this. Suzieq is one way, I'm sure there are others. 
## Suzieq
Try out [Suzieq](https://www.stardustsystems.net/suzieq/), our open source, multivendor tool for network understanding. Suzieq collects operational state in your network and lets you find, validate, and explore your network.

## Conversation

1. How do you explore, find, and understand your network today?
2. Are there things you'd like to investigate that are too hard right now?
3. Try out Suzieq
4. When you've tried out Suzieq, what would you like to help you explore, find, and understand your network?

